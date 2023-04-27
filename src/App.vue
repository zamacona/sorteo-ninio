<script setup>
import { ref, watch, reactive } from 'vue'
import { gsap } from "gsap";

import "animate.css";
import _, { map } from "underscore";
import Upload from "./components/Upload.vue";

const ganadores = ref([]);
const actual = ref('');
const numero = ref('');
const tweened = reactive({
    number: 0
})
const file = ref(null);
const csvIsLoaded = ref('');
const boletos = ref([]);

const ganadoresInput = ref(null);

watch(actual, (n) => {
    if (n == '') return;
    gsap.to(tweened, { duration: 0.7, number: Number(n) || 0 });
});

const save = () => {
    if (numero.value != "") {
        if (exists()) {

            alert('El numero ' + numero.value + ' ya fue seleccionado.');
            return;
        }

        if (actual.value != "") {
            ganadores.value.push(actual.value);
            localStorage.ganadores = JSON.stringify(ganadores.value);
        }
        actual.value = numero.value;
        localStorage.actual = actual.value;
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
    if (actual.value == numero.value || _.find(ganadores.value, function (num) { return num == numero.value; }) != undefined)
        return true;
    else return false;
};

const clean = () => {
    if (localStorage.ganadores) {
        ganadores.value = [];
        localStorage.ganadores = JSON.stringify(ganadores.value);
    }
    if (localStorage.actual) {
        actual.value = "";
        localStorage.actual = actual.value;
    }
    tweened.number = 0;
};

const cleanActual = () => {
    if (localStorage.actual) {
        actual.value = "";
        localStorage.actual = actual.value;
    }
    tweened.number = 0;
};

const loaded = (event) => {
    csvIsLoaded.value = event.split("\n");
    csvIsLoaded.value.forEach((row) => {
        if (row != '') {
            let arr = row.split(",");
            boletos.value.push(arr[0]);
        }
    });
};

const getRandom = () => {
    do {
        let index = getRandomInt(1, boletos.value.length);
        numero.value = boletos.value[index];
        console.log(numero.value);
    } while (exists())
    save();
};

const getRandomInt = (min, max) => {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min) + min);
}

if (localStorage.ganadores) {
    ganadores.value = JSON.parse(localStorage.ganadores);
    if (ganadores.value.length > 0)
        setTimeout(() => {
            const el = ganadoresInput[ganadores.value.length - 1];
            if (el) {
                el.scrollIntoView({ behavior: "smooth" });
            }
        }, 300);
}
if (localStorage.actual) actual.value = localStorage.actual;
</script>

<template>
    <div class="flex flex-col h-screen border-0 border-red-500">
        <TransitionGroup style="background-image: url('escudo-guerrero.png')" class="
            bg-[#343a3f] bg-no-repeat bg-left
            py-3
            pr-3
            pl-96
            border-0 border-teal-500
            flex-none
            h-32
            flex flex-row-reverse flex-nowrap
            gap-x-3
            overflow-auto overflow-y-visible
          " tag="div" name="custom-classes" enter-active-class="animate__animated animate__slideInLeft">
            <template v-if="csvIsLoaded">
                <div v-for="(ganador, index) in ganadores" :key="index" ref="ganadoresInput" class="
                w-40
                flex-none
                border-0 border-blue-500
                grid
                place-items-center
                bg-white
                shadow-md
                rounded-lg
                text-center
              ">
                    <span class="text-6xl">
                        {{ ganador }}
                    </span>
                </div>
            </template>
        </TransitionGroup>
        <div class="border-b-2 border-gray-200"></div>
        <div style="background-image: url('bg-center.jpeg'); border-radius: 50px" class="
            border-0 border-sky-500
            flex-none
            h-[calc(100%-15rem)]
            text-center
            m-3
            bg-no-repeat bg-left-top
            bg-[length:200px_200px]
          ">
            <div class="h-full grid place-items-center" v-if="tweened.number != 0 && csvIsLoaded">
                <span class="text-9xl font-semibold">{{ tweened.number.toFixed(0) }}</span>
                <button class="justify-self-end text-xs bg-red-500 text-white p-1 rounded-sm" @click="cleanActual">
                    Borrar
                </button>
            </div>
        </div>
        <div class="border-0 border-fuchsia-500 m-3 relative h-14">
            <div class="flex absolute right-0">
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
                " type="number" v-model="numero" @keypress.enter="save" />
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
                " @click="save">
                        Agregar
                    </button>
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
                " @click="getRandom">
                        Aleatorio
                    </button>
                </template>
            </div>
        </div>
    </div>
</template>