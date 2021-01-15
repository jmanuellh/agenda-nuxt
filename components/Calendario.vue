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
            v-btn( icon dark @click="limpiarFormEvento()" )
              v-icon mdi-close
            v-toolbar-title Nuevo evento
            v-spacer
            span( v-if="idNuevoEvento == ''" )
              v-toolbar-items
                v-btn(
                  dark
                  text
                  @click="agregarEvento()"
                ) Guardar
            span( v-else )
              v-toolbar-items
                v-btn(
                  dark
                  text
                  @click="actualizarEvento()"
                ) Actualizar
          div(class="ma-6" )
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
        v-model="tipo"
        :items="tipos"
        dense
        outlined
        hide-details
        class="ma-2"
        label="Tipo"
        @change="cambioTipo()"
      )
      //- v-select(
      //-   v-model="mode"
      //-   :items="modes"
      //-   dense
      //-   outlined
      //-   hide-details
      //-   label="Modo"
      //-   class="ma-2")
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
          @click:event="showEvent"
        )
    v-menu(
      v-model="selectedOpen"
      :close-on-content-click="false"
      :activator="selectedElement"
      offset-x
    )
      v-card(
        color="grey lighten-4"
        min-width="350px"
        flat
      )
        v-toolbar(
          :color="selectedEvent.color"
          dark
        )
          v-btn(
            icon
            @click="editarEvento()"
          )
            v-icon mdi-pencil
          v-toolbar-title( v-html="selectedEvent.name" )
          v-spacer
          v-btn( icon dark @click="selectedOpen = false" )
            v-icon mdi-close
        v-card-text
          span( v-html="selectedEvent.details" )
        v-card-actions
          v-btn(
            text
            color="secondary"
            @click="selectedOpen = false"
          ) Cancel
          v-spacer
          v-btn(
            text
            color="error"
            @click="eliminarEvento()"
          ) Eliminar
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
      tipo: 'Mensual',
      tipos: ['Mensual', 'Semanal', 'Diario', '4 días'],
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
        dateEnd: '',
        timeEnd: ''
      },
      idNuevoEvento: '',
      selectedEvent: {
        color: '',
        end: '',
        name: '',
        start: ''
      },
      selectedElement: null,
      selectedOpen: false,
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
      this.limpiarFormEvento()

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
    cambioTipo() {
      switch(this.tipo) {
        case 'Mensual':
          this.type = 'month'
          break
        case 'Semanal':
          this.type = 'week'
          break
        case 'Diario':
          this.type = 'day'
          break
        case '4 días':
          this.type = '4day'
          break
      }
    },
    showEvent ({ nativeEvent, event }: { nativeEvent: any; event: any}) {
      const open = () => {
        this.selectedEvent = event
        this.selectedElement = nativeEvent.target
        setTimeout(() => {
          this.selectedOpen = true
        }, 10)
      }

      if (this.selectedOpen) {
        this.selectedOpen = false
        setTimeout(open, 10)
      } else {
        open()
      }

      nativeEvent.stopPropagation()
    },
    async eliminarEvento() {
      let query = this.$fire.firestore.collection('calendario_marisol')
        .where('start','==',this.selectedEvent.start)
        .where('end','==',this.selectedEvent.end);      
      let querySnapshot = await query.get();

      querySnapshot.forEach(async function(doc) {
        await doc.ref.delete();
      });
      this.obtenerEventos()
      this.selectedOpen = false
    },
    async editarEvento() {
      let query = this.$fire.firestore.collection('calendario_marisol')
        .where('start','==',this.selectedEvent.start)
        .where('end','==',this.selectedEvent.end);    
      let querySnapshot = await query.get();

      querySnapshot.forEach(async doc => {
        let nuevoEvento = await doc.data();
        this.nuevoEvento = {
          name: nuevoEvento.name,
          dateStart: nuevoEvento.start.split(' ')[0],
          timeStart: nuevoEvento.start.split(' ')[1],
          dateEnd: nuevoEvento.end.split(' ')[0],
          timeEnd: nuevoEvento.end.split(' ')[1]
        }
        this.idNuevoEvento = doc.id
      });
      this.dialog = true
      this.selectedOpen = false
    },
    async actualizarEvento() {
      let nuevoEvento = {
        name: this.nuevoEvento.name,
        start: this.nuevoEvento.dateStart +" "+ this.nuevoEvento.timeStart,
        end: this.nuevoEvento.dateEnd +" "+ this.nuevoEvento.timeEnd,
        color: 'blue'
      }

      await this.$fire.firestore.collection("calendario_marisol").doc(this.idNuevoEvento).update(nuevoEvento)
      this.obtenerEventos()
      this.limpiarFormEvento()
    },
    limpiarFormEvento() {
      this.nuevoEvento = {
        name: '',
        dateStart: '',
        timeStart: '',
        dateEnd: '',
        timeEnd: ''
      }
      this.idNuevoEvento = ''
      this.dialog=false
    }
  }
})
</script>

<style scoped>
</style>