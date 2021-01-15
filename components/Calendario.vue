<template lang="pug">
  div
    div(
      class="ma-4"
      align="center"
    )
      v-dialog(
        v-model="dialog"
        fullscreen
        hide-overlay
        transition="dialog-bottom-transition"
      )
        template(v-slot:activator="{ on, attrs }")
          v-btn(
            color="primary"
            dark
            v-bind="attrs"
            v-on="on"
          ) Nuevo evento
        v-card
          v-toolbar( dark color="primary" )
            v-btn( icon dark @click="dialog=false" )
              v-icon mdi-close
            v-toolbar-title Nuevo evento
            v-spacer
            v-toolbar-items
              v-btn(
                dark
                text
                @click="agregarEvento()"
              ) Guardar
          v-form(
            ref="form"
          )
            v-text-field(
              v-model="nuevoEvento.name"
              label="Nombre"
              type="text" 
            )
            v-text-field(
              v-model="nuevoEvento.dateStart"
              label="Fecha de inicio"
              type="date" 
            )
            v-text-field(
              v-model="nuevoEvento.timeStart"
              label="Hora de inicio"
              type="time"
            )
            v-text-field(
              v-model="nuevoEvento.dateEnd"
              label="Fecha de fin"
              type="date" 
            )
            v-text-field(
              v-model="nuevoEvento.timeEnd"
              label="Hora de fin"
              type="time"
            )
    div(class="d-flex")
      v-btn(
        icon
        class="ma-2"
        @click="$refs.calendar.prev()"
      )
        v-icon mdi-chevron-left 
      v-spacer
      v-select(
        v-model="type"
        :items="types"
        dense
        outlined
        hide-details
        class="ma-2"
        label="Tipo"
      )
      v-select(
        v-model="mode"
        :items="modes"
        dense
        outlined
        hide-details
        label="Modo"
        class="ma-2")
      v-spacer
      v-btn(
        icon
        class="ma-2"
        @click="$refs.calendar.next()"
      )
        v-icon mdi-chevron-right
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
      types: ['month', 'week', 'day', '4day'],
      events: [] as any,
      mode: 'stack',
      modes: ['stack', 'column'],
      nombre: 'nombre',
      arrayPrueba: [] as any,
      dialog: false,
      nuevoEvento: {
        name: '',
        dateStart: '',
        timeStart: '',
        dateTimeStart: '',
        dateEnd: '',
        timeEnd: '',
        dateTimeEnd: ''
      }
    }
  },
  async mounted() {
    this.obtenerEventos()
  },
  methods: {
    getEventColor (event: any) {
      return event.color
    },
    async agregarEvento() {
      let nuevoEvento = {
        name: this.nuevoEvento.name,
        start: this.nuevoEvento.dateStart +" "+ this.nuevoEvento.timeStart,
        end: this.nuevoEvento.dateEnd +" "+ this.nuevoEvento.timeEnd,
        color: 'blue'
      }

      await this.$fire.firestore.collection('calendario_marisol').add(nuevoEvento)
      this.obtenerEventos()
      this.dialog = false

      // this.events.push({
      //   name: 'evento',
      //   start: this.$moment().format('YYYY-MM-DD hh:mm'),
      //   end: this.$moment().add(2, 'hours').format('YYYY-MM-DD hh:mm'),
      //   color: 'blue'
      // })
    },
    async obtenerEventos() {
      this.events = (await this.$fire.firestore.collection('calendario_marisol').get()).docs.map(doc => doc.data())
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