<script setup>
import { ref } from 'vue'
const emit = defineEmits(['loaded', 'data']);

const totalReaded = ref("0%");
const loadingXMLFiles = ref(0); //0 no, 1 leyendo, 2 lectura finalizada

const file = ref(null)

const clean = () => {
    totalReaded.value = "0%";
    loadingXMLFiles.value = 0;
};

const loadXml = () => {
    let total = file.value.files.length;
    for (let i = 0; i < total; i++) {
        let loadedFile = file.value.files[i];
        if (loadedFile) {
            getAsText(loadedFile, total, i + 1);
        }
    }
    loadingXMLFiles.value = 2;
};

const getAsText = (file, total, index) => {
    let reader = new FileReader();
    reader.totalFiles = total;
    reader.indexFile = index;
    reader.fileName = file.name;
    // Read file into memory as UTF-8
    reader.readAsText(file, "UTF-8");
    // Handle progress, success, and errors
    reader.onprogress = updateProgress;
    reader.onload = loaded;
    reader.onerror = errorHandler;
};

const updateProgress = (evt) => {
    if (evt.lengthComputable) {
        // evt.loaded and evt.total are ProgressEvent properties
        var loaded = evt.loaded / evt.total;
        if (loaded < 1) {
            // Increase the prog bar length
        }
    }
};

const loaded = (evt) => {
    // Obtain the read file data
    var fileString = evt.target.result;
    var fileName = evt.target.fileName;
    var fileNameArr = fileName.split("_");

    let total =
        Math.round(
            ((evt.target.indexFile * 100) / evt.target.totalFiles) * 100
        ) / 100;
    if (total < 100) {
        loadingXMLFiles.value = 1;
        totalReaded.value = total + "%";
    } else if (total >= 100) {
        totalReaded.value = "100%";
        loadingXMLFiles.value = 2;
        emit("loaded", fileString);
    }
};

const errorHandler = (evt) => {
    if (evt.target.error.name == "NotReadableError") {
        // The file could not be read
    }
};
</script>

<template>
    <form>
        <div>
          <div>
            <div class="mt-0 flex items-center">
              <label
                for="file-upload"
                class="
                  relative
                  cursor-pointer
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
                "
              >
                <span class="align-bottom text-sm"
                  >{{
                    loadingXMLFiles == 0
                      ? "Seleccionar archivo"
                      : loadingXMLFiles == 1
                      ? "Leyendo archivo .csv " + totalReaded
                      : "Archivo cargado " + totalReaded
                  }}
                </span>
                <input
                  v-on:change="loadXml()"
                  id="file-upload"
                  name="file-upload"
                  ref="file"
                  multiple="multiple"
                  type="file"
                  class="sr-only"
                  accept=".csv"
                />
              </label>
            </div>
          </div>
        </div>
      </form>
</template>