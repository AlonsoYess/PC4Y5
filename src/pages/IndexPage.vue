<template>
  <q-page class="q-pa-md">
    <q-form @submit="consultarApi">
      <div class="row">
        <div class="col-4">
          <q-input v-model="startDate" type="date" label="Fecha de inicio" :max="today"/>
        </div>
        <div class="col-4">
          <q-input v-model="endDate" type="date" label="Fecha de fin" :max="today"/>
        </div>
      </div>
      <q-btn type="submit" label="Consultar" color="primary" class="q-mt-md" />
    </q-form>

    

    
      <div v-if="loading" class="loading-container">
        <q-spinner-cube size="100px" color="primary" />
      </div>
    


    <div v-if="!loading && data.length === 0 && hizoConsulta" class="q-mt-md">
      <q-banner type="warning" inline-actions>
        <template v-slot:avatar>
          <q-icon name="warning" />
        </template>
        No se encontraron registros para el rango de fechas seleccionado.
      </q-banner>
    </div>

    <q-card v-for="item in data" :key="item.date" class="q-mt-md">
      <q-card-section>
        <div class="text-h6">{{ item.title }}</div>
        <div class="text-subtitle1">{{ item.date }}</div>
      </q-card-section>
      <q-img :src="item.url" :alt="item.title" v-if="item.media_type === 'image'" />
      <q-card-section v-if="item.media_type === 'video'">
        <q-btn :href="item.url" target="_blank" label="Ver video" color="primary" />
      </q-card-section>
      <q-card-section>
        <div>{{ item.explanation }}</div>
      </q-card-section>
    </q-card>


  </q-page>
</template>

<script>
import axios from 'axios';
import { Notify } from 'quasar';

export default {
  data() {

    const hoy = new Date();
    const ayer = new Date(hoy);
    ayer.setDate(ayer.getDate() - 1);

    const formatoFecha = (date) => date.toISOString().split('T')[0];

    return {
      startDate: formatoFecha(ayer),
      endDate: formatoFecha(hoy),
      today: formatoFecha(hoy),
      data: [],
      hizoConsulta : false,
      loading: false
    };
  },
  methods: {

    validarFormulario() {
      if (!this.startDate) {
        Notify.create({
          type: 'negative',
          message: 'La fecha de inicio no puede estar vacía.',
        });
        return false;
      }
      if (!this.endDate) {
        Notify.create({
          type: 'negative',
          message: 'La fecha de fin no puede estar vacía.',
        });
        return false;
      }
      if (this.startDate > this.endDate) {
        Notify.create({
          type: 'negative',
          message: 'La fecha de inicio no puede ser mayor que la fecha de fin.',
        });
        return false;
      }
      return true;
    },

    async consultarApi() {

      if (!this.validarFormulario()) {
        return;
      }

      this.loading = true;

      try {
        const response = await axios.get('https://api.nasa.gov/planetary/apod', {
          params: {
            api_key: 'Fg3fbLeL5eN8xZX8UL9UI9helJkodRXIt6Yawb9U',
            start_date: this.startDate,
            end_date: this.endDate,
          },
        });
        this.data = response.data;
        this.hizoConsulta = true;

        if (this.data.length === 0) {
          Notify.create({
            type: 'warning',
            message: 'No se encontraron registros para el rango de fechas seleccionado.',
          });
        }
      
      } catch (error) {
      
        Notify.create({
            type: 'negative',
            message: 'Error al recuperar datos: ' + error.message,
          });

      }finally {
        this.loading = false; // Finalizar carga
      }
    },
  },
};
</script>

<style>
.q-page {
  max-width: 800px;
  margin: 0 auto;
}
.q-card {
  max-width: 600px;
}

.loading-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(255, 255, 255, 0.8); /* Fondo semitransparente */
  z-index: 1000; /* Asegurarse de que esté por encima de otros elementos */
}


</style>
