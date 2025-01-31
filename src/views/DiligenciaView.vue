<script lang="ts" setup>
import useDatosDiligencia from '@/composables/useDatosDiligencia';
import { getUpperCase } from '@/helpers/stringUtils';
import { computed, ref, watch } from 'vue';
import useNewActuacion from '../composables/useNewActuacion';
import useSaveData from '../composables/useSaveData';
import useItem from '@/composables/useItems';
import { useRouter } from 'vue-router';

interface Props{
  actuacion:string;
}
const props = defineProps<Props>()
const actuacionRef = ref(props.actuacion);
const router = useRouter()
const { 
  processedText, 
  primeradiligencia,
  processedHeaderText,
  isEditingHeader,
  headerContainer,
  headerTextComputed,
  relato
} = useDatosDiligencia(actuacionRef);

const {isEditedHeader} = useNewActuacion()

const { saveData} = useSaveData()
const {afectados,efectos,fechaUbicacion,intervinientes,vinculados} = useItem()
const toggleHeader = () => {
  if (isEditingHeader.value) {
    
    headerContainer.value = headerTextComputed.value; // Usar headerTextComputed permite reflejar los cambios
    isEditedHeader.value = true; // Se mueve aquí para reflejar que ahora hay un valor editado
  }else{
    if (headerContainer.value === '') {
      headerContainer.value = processedHeaderText.value;
    }
  }
  isEditingHeader.value = !isEditingHeader.value;
};
const handleSave = ()=>{
  const data={
    afectados:afectados.value,
    vinculados:vinculados.value,
    fechaUbicacion:fechaUbicacion.value,
    efectos:efectos.value,
    personalInterviniente:intervinientes.value ?? []
  }
  saveData(data)
  /* tengo que resetear todo */
  router.push({ name: 'actuaciones' });
}
watch(() => props.actuacion, (newValue) => {
  actuacionRef.value = newValue;
});

</script>
<template>
    <div class="surface-section px-2 py-5 md:px-6 lg:px-8 w-full">
      <div class="text-700 text-justify">
        <div class="flex justify-content-between align-items-center mb-5">
          <div class="font-medium text-3xl text-900">{{ primeradiligencia ? getUpperCase(primeradiligencia.titulo) : '' }}</div>
          <div>
            <Button label="Previsualizar" class="p-button-rounded mr-2" />
            <Button label="Registrar" class="p-button-rounded " @click="handleSave" severity="warning"/>
          </div>
        </div>

        <div class="text-500 mb-3">Este diligencia es ....</div>
        <ul class="list-none p-0 m-0 w-full">
          <li class="flex align-items-center py-3 px-2 border-top-1 surface-border" style="justify-content: space-between;">
            <div v-if="!isEditingHeader && !isEditedHeader" v-html="processedText.header"></div>
            <div v-else-if="!isEditingHeader && isEditedHeader" >{{ headerContainer }}</div>
            <Textarea v-if="isEditingHeader" v-model="headerTextComputed" autoResize class="w-full" />

            <div>
              <Button
                :class="{'ml-3': true, 'p-button-rounded': true, 'p-button-warning': isEditingHeader, 'p-button-help': !isEditingHeader}"
                :icon="isEditingHeader ? 'pi pi-check' : 'pi pi-pencil'"
                @click="toggleHeader"
              ></Button>
            </div>
          </li>
          <li class="py-3 px-2 border-top-1 surface-border">
            <div class="text-500 font-medium mb-2">Relato</div>
            <div> 
              <Textarea rows="10" class="w-full" v-model="relato"/>
            </div>
          </li>
          <li class="flex align-items-center py-3 px-2 border-top-1 surface-border" style="justify-content: space-between;">
            <div  v-html="processedText.footer"  style="flex-grow: 1;"></div>
            <div>
              <Button class="ml-3"  icon="pi pi-pencil" rounded></Button>
            </div>
          </li>
        </ul>
      </div>
  </div>
</template>
