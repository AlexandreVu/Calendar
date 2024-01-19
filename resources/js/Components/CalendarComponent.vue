<template>
  <div>
      <div>
        <div>
          <form @submit.prevent>
            <div>
              <label for="event_name">Event Name : </label>
              <input type="text" id="event_name" v-model="newEvent.event_name">
            </div>
            <div>
              <div>
                <div>
                  <label for="event_start">Start Date : </label>
                  <input
                    type="date"
                    id="event_start"
                    class="form-control"
                    v-model="newEvent.event_start"
                  >
                </div>
              </div>
              <div>
                <div>
                  <label for="event_end">End Date : </label>
                  <input type="date" id="event_end" class="form-control" v-model="newEvent.event_end">
                </div>
              </div>
              <div>
                <button @click="addNewEvent">Save Event</button>
              </div>
            </div>
          </form>
        </div>
        <br>
        <div>
          <FullCalendar :options="calendarOptions"/>
        </div>
    </div>
  </div>
</template>

<script>
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import axios from "axios"

export default {
  components: {
    FullCalendar
  },
  data() {
    return {
      calendarOptions: {
        plugins: [ dayGridPlugin, interactionPlugin ],
        initialView: 'dayGridMonth',
        eventClick: this.showEvent,
        events: []
      },
      indexToUpdate: "",
      newEvent: {
        title: "",
        event_start: "",
        event_end: ""
      }
    }
  },
  created() {
    this.getEvents();
  },
  methods: {
    addNewEvent() {
      axios
        .post("/addCalendarEvent", {
          ...this.newEvent
        })
        .then(data => {
          this.getEvents();
          this.resetForm();
        })
        .catch(err =>
          console.log("Unable to add new event!", err.response.data)
        );
    },
    showEvent(arg) {
      const { id, title, start, end } = this.calendarOptions.events.find(
        event => event.id === +arg.event.id
      );
      this.indexToUpdate = id;
      this.newEvent = {
        event_name: title,
        event_start: start,
        event_end: end
      };
    },
    getEvents() {
      axios
        .get("/getCalendarEvents")
        .then(resp => (this.calendarOptions.events = resp.data.data))
        .catch(err => console.log(err.response.data));
    },
    resetForm() {
      Object.keys(this.newEvent).forEach(key => {
        return (this.newEvent[key] = "");
      });
    }
  },
  watch: {
    indexToUpdate() {
      return this.indexToUpdate;
    }
  }
}
</script>