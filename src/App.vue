<script setup>
import { ref, watch, reactive, computed } from 'vue'
import { gsap } from "gsap";

import "animate.css";
import _, { map } from "underscore";
import Upload from "./components/Upload.vue";

const ganadores = ref([]);
const actual = ref(null);


const numero = ref('');
const tweened = reactive({
    number: 0
})
const file = ref(null);
const csvIsLoaded = ref('');
const boletos = ref([]);

const ganadoresInput = ref(null);

const from = ref(0);
const via = ref(0);
const to = ref(0);

const colorsFrom = [
    'from-blue-500', 'from-red-500', 'from-green-500', 'from-pink-500', 'from-orange-500', 'from-amber-500', 'from-yellow-500', 'from-lime-500', 'from-emerald-500', 'from-teal-500', 'from-cyan-500', 'from-sky-500', 'from-indigo-500', 'from-violet-500', 'from-purple-500', 'from-fuchsia-500', 'from-rose-500',
];
const colorsVia = [
    'via-blue-500', 'via-red-500', 'via-green-500', 'via-pink-500', 'via-orange-500', 'via-amber-500', 'via-yellow-500', 'via-lime-500', 'via-emerald-500', 'via-teal-500', 'via-cyan-500', 'via-sky-500', 'via-indigo-500', 'via-violet-500', 'via-purple-500', 'via-fuchsia-500', 'via-rose-500',
];
const colorsTo = [
    'to-blue-500', 'to-red-500', 'to-green-500', 'to-pink-500', 'to-orange-500', 'to-amber-500', 'to-yellow-500', 'to-lime-500', 'to-emerald-500', 'to-teal-500', 'to-cyan-500', 'to-sky-500', 'to-indigo-500', 'to-violet-500', 'to-purple-500', 'to-fuchsia-500', 'to-rose'
];

watch(actual, (n) => {
    if (n?.folio == null) return;
    gsap.to(tweened, { duration: 0.7, number: Number(n.folio) || 0 });
})


const classes = computed(() => {
    return colorsFrom[from.value] + ' ' + colorsVia[via.value] + ' ' + colorsTo[to.value];
})

const loaded = (event) => {
    csvIsLoaded.value = event.split("\n");
    csvIsLoaded.value.forEach((row) => {
        if (row != '') {
            let arr = row.split(',');
            boletos.value.push({
                folio: arr[0].trim(),
                nombre: arr[1]?.trim()
            });
        }
    });
    if (ganadores.value.length > 0)
        setTimeout(() => {
            const el = ganadoresInput.value[ganadores.value.length - 1];
            if (el) {
                el.scrollIntoView({ behavior: "smooth" });
            }
        }, 300);
};

const manual = () => {
    numero.value = getBoleto(numero.value);
    if (exists()) {
        alert('El numero ' + numero.value.folio + ' ya fue seleccionado.');
        return;
    }
    save();
}

const save = () => {
    if (numero.value != "") {
        if (exists()) {
            alert('El numero ' + numero.value.folio + ' ya fue seleccionado.');
            return;
        }

        if (actual.value != null) {
            ganadores.value.push({ boleto: actual.value, classes: classes.value });
            localStorage.ganadores = JSON.stringify(ganadores.value);
        }

        from.value = getRandomInt(0, colorsFrom.length);
        via.value = getRandomInt(0, colorsVia.length);
        to.value = getRandomInt(0, colorsTo.length);

        actual.value = numero.value;
        localStorage.actual = JSON.stringify(actual.value);

        numero.value = "";
        if (ganadores.value.length > 0)
            setTimeout(() => {
                const el = ganadoresInput.value[ganadores.value.length - 1];
                if (el) {
                    el.scrollIntoView({ behavior: "smooth" });
                }
            }, 300);
    }
};

const exists = () => {
    if (actual.value?.folio == numero.value.folio
        || (_.find(ganadores.value, function (ganador) { return ganador.boleto.folio == numero.value.folio; }) != undefined)
    )
        return true;
    else return false;
};

const clean = () => {
    if (localStorage.ganadores) {
        ganadores.value = [];
        localStorage.ganadores = JSON.stringify(ganadores.value);
    }
    if (localStorage.actual) {
        actual.value = null;
        localStorage.actual = actual.value;
    }
    tweened.number = 0;
};

const cleanActual = () => {
    if (localStorage.actual) {
        actual.value = null;
        localStorage.actual = actual.value;
    }
    tweened.number = 0;
};



const getBoleto = (folio) => {
    if (!csvIsLoaded.value) return;
    let boleto = _.find(boletos.value, function (boleto) {
        return boleto.folio == folio;
    });
    if (boleto == undefined) {
        return {
            folio: folio,
            nombre: ""
        };
    }
    return boleto;
}

const getNombre = (folio) => {
    if (!csvIsLoaded.value) return;
    let boleto = _.find(boletos.value, function (boleto) {
        return boleto.folio == folio;
    });
    if (boleto == undefined) {
        return "";
    }
    return boleto.nombre;
}

const mouseup = () => {
    numero.value = getBoleto(300);
    save();
};

const getRandom = () => {
    do {
        let index = getRandomInt(1, boletos.value.length);
        numero.value = boletos.value[index];
    } while (exists())
    save();
};


const getRandomInt = (min, max) => {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min) + min);
}


//carga datos almacenados
if (localStorage.ganadores) ganadores.value = JSON.parse(localStorage.ganadores);
if (localStorage.actual) {
    actual.value = JSON.parse(localStorage.actual);
    from.value = getRandomInt(0, colorsFrom.length);
    via.value = getRandomInt(0, colorsVia.length);
    to.value = getRandomInt(0, colorsTo.length);
}
</script>

<template>
    <div class="flex flex-col h-screen border-0 border-red-500">
        <div class="pl-96 bg-[#343a3f] bg-no-repeat bg-left" style="background-image: url('escudo-guerrero.png')">
            <TransitionGroup class="
            py-3
            pr-3
            border-0 border-teal-500
            flex-none
            h-32
            flex flex-row-reverse flex-nowrap
            gap-x-3
            overflow-auto overflow-y-visible
            scrollbar-thin scrollbar-thumb-gray-400 scrollbar-track-gray-300
          " tag="div" name="custom-classes" enter-active-class="animate__animated animate__slideInLeft">
                <template v-if="csvIsLoaded">
                    <div v-for="(ganador, index) in ganadores" :key="index" ref="ganadoresInput" class="
                w-48
                flex-none
                border-0 border-blue-500
                grid
                place-items-center
                bg-white
                shadow-md
                rounded-lg
                text-center
              ">
                        <span class='font-mono text-4xl text-transparent bg-clip-text bg-gradient-to-br' :class="ganador.classes">
                            {{ ganador.boleto.folio }}
                        </span>
                        <span class='font-mono text-base text-transparent bg-clip-text bg-gradient-to-br' :class="ganador.classes">
                            {{ ganador.boleto.nombre }}
                        </span>
                    </div>
                </template>
            </TransitionGroup>
        </div>
        <div class="border-b-2 border-gray-200"></div>
        <div style="background-image: url('bg-center-2023.jpg'); border-radius: 50px" class="
            border-0 border-sky-500
            flex-none
            h-[calc(100%-15rem)]
            text-center
            m-3
            bg-no-repeat bg-left-top
            bg-[length:350px_300px]
          ">
            <div class="h-full grid place-items-center">
                <span class='font-mono text-[250px] font-semibold text-transparent bg-clip-text bg-gradient-to-br' :class="[tweened.number != 0 ? classes : '']" v-if="csvIsLoaded">
                    {{ tweened.number != 0 ? tweened.number.toFixed(0) : '0' }}
                </span>
                <span class='font-mono text-5xl font-semibold text-transparent bg-clip-text bg-gradient-to-br' :class="[tweened.number != 0 ? classes : '']" v-if="csvIsLoaded">
                    {{ getNombre(tweened.number.toFixed(0)) }}
                </span>
                <button v-if="csvIsLoaded" @click="getRandom" @mouseup.right="mouseup" @contextmenu.prevent class="
                inline-block
                ml-4
                px-6
                py-2
                text-white
                font-medium
                text-4xl
                leading-tight
                uppercase
                rounded
                hover:bg-black hover:bg-opacity-5
                focus:outline-none focus:ring-0
                transition
                duration-150
                ease-in-out
                tracking-widest
                bg-gradient-to-r from-green-400 to-blue-500 hover:from-pink-500 hover:to-yellow-500
              ">
                    G I R A R
                </button>
                <!-- <button class="justify-self-end text-xs bg-red-500 text-white p-1 rounded-sm" @click="cleanActual">
                    Borrar
                </button> -->
            </div>
        </div>
        <div class="border-0 border-fuchsia-500 m-3 relative h-14">
            <div class="flex absolute left-0">
                <button @click="clean" v-if="csvIsLoaded">Borrar</button>
                <Upload @loaded="loaded"></Upload>
                <template v-if="csvIsLoaded">
                    <input class="
                  ml-4
                  px-3
                  py-1.5
                  text-base
                  font-normal
                  text-gray-700
                  bg-white bg-clip-padding
                  border border-solid border-gray-300
                  rounded
                " type="number" v-model="numero" @keypress.enter="manual" />
                    <button class="
                  inline-block
                  ml-4
                  px-6
                  py-2
                  border-0 border-blue-600
                  text-blue-600
                  font-medium
                  text-xs
                  leading-tight
                  uppercase
                  rounded
                  hover:bg-black hover:bg-opacity-5
                  focus:outline-none focus:ring-0
                  transition
                  duration-150
                  ease-in-out
                " @click="manual">
                        Agregar
                    </button>
                </template>
            </div>
        </div>
    </div>
</template>
<style setup>
::-webkit-scrollbar {
    height: 6px;
}

::-webkit-scrollbar-thumb {
    background-color: #718096;
}

::-webkit-scrollbar-track {
    background-color: #CBD5E0;
}
</style>