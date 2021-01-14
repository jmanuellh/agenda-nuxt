<template lang="pug">
  div
    div(class="ma-4" align="center")
      v-btn(
        color="primary"
      ) Agregar evento
    div
      v-sheet(height="30rem")
        v-calendar(
          locale="es"
          ref="calendar"
          v-model="value"
          :weekdays="weekday"
          :type="type"
          :events="events"
          :event-overlap-mode="mode"
          :event-overlap-threshold="30"
          :event-color="getEventColor"
          :short-weekdays="false"
        )
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  data() {
    return {
      value: '',
      weekday: [1, 2, 3, 4, 5, 6, 0],
      type: 'month',
      events: [] as any,
      mode: 'stack',
      nombre: 'nombre',
      arrayPrueba: [] as any
    }
  },
  async mounted() {
    this.obtenerEventos()
  },
  methods: {
    getEventColor (event: any) {
      return event.color
    },
    agregarEvento() {
      this.events.push({
        name: 'evento',
        start: this.$moment().format('YYYY-MM-DD hh:mm'),
        end: this.$moment().add(2, 'hours').format('YYYY-MM-DD hh:mm'),
        color: 'blue'
      })
    },
    async obtenerEventos() {
      // this.events = (await this.$fire.firestore.collection('calendario_marisol').get()).docs.map(doc => doc.data())
      // this.$fire.firestore.collection('calendario_marisol').get().then(r => {
      //   console.log(r.docs.map(doc => doc.data()))
      //   // r.forEach(function(doc) {
      //   //     // doc.data() is never undefined for query doc snapshots
      //   //     console.log(doc.data());
      //   // });
      // })
    },
  }
})
</script>

<style scoped>
</style>