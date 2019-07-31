<template lang="html">
  <div class="">
<!-- top bar -->
    <div class="w3-cell-row">
      <div class="w3-cell">
        <button type="button" class="w3-block w3-btn w3-deep-purple" name="button" v-on:click="subtractMonth"><i class="fas fa-chevron-left"></i></button>
      </div>
      <div class="w3-cell w3-center w3-teal">
        <h4 class="line w3-margin-left w3-margin-right">{{month}} {{year}}</h4>
      </div>
      <div class="w3-cell">
        <button type="button" class="w3-block w3-btn w3-deep-purple" name="button" v-on:click="addMonth"><i class="fas fa-chevron-right" ></i></button>
      </div>
    </div>
<!-- /top bar -->
<!-- calendar -->
    <div class="w3-container w3-padding-16">
      <div class="w3-quarter w3-hide-medium w3-hide-small">
        <div class="w3-row"> 
          <div class="w3-cell">
            <button type="button" class="w3-btn" name="button" v-on:click="subtractMonth"><i class="fas fa-chevron-left"></i></button>
          </div>
          <div class="w3-cell">
            <p>{{month}} {{year}}</p>
          </div>
          <div class="w3-cell">
            <button type="button" class="w3-btn" name="button" v-on:click="addMonth"><i class="fas fa-chevron-right" ></i></button>
          </div>
        </div>
        <div class="w3-row">
          <ul class="weekdays resize">
            <li class="w3-blue-grey w3-display-container default" :key="index" v-for="(day, index) in days"><div class="w3-display-middle">{{day}}</div></li>
          </ul>
          <ul class="dates resize leftClear">
            <li :key="'blankb' + index + month" v-for="(blank, index) in firstDayOfMonth" class="default">&nbsp;</li>
            <li :key="'dateb' + date + index + month" v-for="(date, index) in daysInMonth" class="w3-display-container pointer" :class="{'w3-blue': date == initialDate &amp;&amp; month == initialMonth && year == initialYear, 'w3-grey': date == currentDate && date !== initialDate, 'w3-grey': date == currentDate && month !== initialMonth}" @click="selectDate(year, month, date)" >
              <div class="w3-display-middle CallendarNumber">{{date}}</div>
            </li>
          </ul>
        </div>
      </div>
<!-- /calendar -->
<!-- actual agenda -->
      <div class="w3-rest">
        <div class="w3-cell">
          <div class="w3-cell">
            <button type="button" class="w3-btn" name="button" v-on:click="subtractWeek"><i class="fas fa-chevron-left"></i></button>
          </div>
          <div class="w3-cell">
            <p>Change week</p>
          </div>
          <div class="w3-cell">
            <button type="button" class="w3-btn" name="button" v-on:click="addWeek"><i class="fas fa-chevron-right" ></i></button>
          </div>
        </div>
        <div class="w3-col default" style="width:9%">
          <div class="w3-row w3-center">{{timezone}}</div>
          <div class="w3-row w3-center" v-for="j in 23" :key="'hour'+j" :class="{'hourHeight': j == 1, 'agendaHeight': j > 1}" >
            <span v-show="j < 10">0</span>{{j}}:00
          </div>
        </div>
        <div v-for="(day, index) in daysAbrev" :key="'days'+index" class="w3-col" style="width:12.5%">
          <div class="w3-row default w3-border w3-center" :class="{'w3-grey':numberDaysWeek[index] == currentDate}">
            <span>{{day}} {{getDay(numberDaysWeek[index])}}</span>
          </div>
          <div class="w3-row pointer agendaHeight appointPad" v-for="m in 24" :key="'hour'+m" @click="openModal(numberDaysWeek[index], m, true)" :class="{'w3-blue w3-border-left':appointmentInDay(numberDaysWeek[index], m)[1] !== 'new', 'w3-border':appointmentInDay(numberDaysWeek[index], m)[1] == 'new'}">
              <p class="margin-0" :class="{'w3-tooltip': appointmentInDay(numberDaysWeek[index], m)[0].notes}">
                {{appointmentInDay(numberDaysWeek[index], m)[0].title}}
                <span class="w3-text w3-pale-blue padding-note" v-if="appointmentInDay(numberDaysWeek[index], m)[0].notes">
                  {{unescape(appointmentInDay(numberDaysWeek[index], m)[0].notes)}}
                </span>
              </p>
          </div>
        </div>
      </div>
    </div>
<!-- /actual agenda -->
<!-- modal -->
    <div id="id01" class="w3-modal" v-if="modalCheck !== 'empty'">
      <div class="w3-modal-content">
        <div class="w3-container">
          <span class="w3-button w3-display-topright" @click="reset()">&times;</span>
          <form @submit.prevent="appointmentToJson(modalCheck)">
            <div class="w3-row">
              <input type="text" v-model="appointmentDesc" placeholder="Add a title" required class="w3-input w3-center w3-bottombar w3-border-blue w3-section w3-round"/>
            </div>
            <div class="w3-row">
              <input type="date" class="w3-input w3-border w3-quarter" v-model="dateModal"/>
              <select v-model.number="startHour" required class="w3-select w3-border w3-quarter">
                <option disabled value="">Select start hour</option>
                <option v-for="hs in 24" :key="'selecthourstart'+hs" v-bind:value="hs-1"><span v-if="hs<=10">0</span>{{hs - 1}}:00</option>
              </select>
              <select v-model.number="endHour" class="w3-select w3-quarter w3-border">
                <option disabled value="">Select end hour</option>
                <option v-for="he in 24" :key="'selecthourend'+he" v-bind:value="he"><span v-if="he<=10">0</span>{{he}}:00</option>
              </select>
              <input type="date" class="w3-input w3-border w3-quarter" v-model="dateModal2"/>
            </div>
            <div class="w3-section">
              <textarea type="text" class="w3-input w3-border" placeholder="notes" style="max-width:100%" v-model="appointmentNotes"/>
            </div>
            <div v-if="modalCheck == 'new'" class="w3-section">
              <button type="submit" class="w3-btn w3-blue w3-right w3-margin-bottom">Submit</button>
            </div>
            <div v-else class="w3-section">
              <button class="w3-btn w3-red w3-right w3-margin-bottom" @click="deleteAppointment(modalCheck)">
                <i class="far fa-trash-alt"></i>
              </button>
              <button type="submit" class="w3-btn w3-blue w3-right w3-margin-bottom">Edit</button>
            </div>
          </form>
        </div>
      </div>
    </div>
<!-- /modal -->
  </div>

</template>

<script>
import moment from "moment";
import momentTimezone from 'moment-timezone';
import validator from 'validator';

export default {
  name: "HelloWorld",
  data() {
    return {
      today: moment(),
      timezone: momentTimezone.tz(momentTimezone.tz.guess()).zoneName(),
      dateContext: moment(),
      dateModal: moment().format("YYYY-MM-DD"),
      dateModal2: moment().format("YYYY-MM-DD"),
      days: ["S", "M", "T", "W", "T", "F", "S"],
      daysAbrev: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"],
      modalCheck: "empty",
      startHour: "",
      endHour: "",
      appointmentDesc: "",
      appointmentsList: [],
      appointmentNotes: ""
    };
  },
  computed: {
    year() {
      return this.dateContext.format("Y");
    },
    month() {
      return this.dateContext.format("MMMM");
    },
    monthIndex() {
      return this.dateContext.get("month");
    },
    daysInMonth() {
      return this.dateContext.daysInMonth();
    },
    currentDate() {
      return this.dateContext.get("date");
    },
    trueDate() {
      return this.today.get("date");
    },
    currentDayOfWeek() {
      return this.dateContext.day();
    },
    firstDayOfMonth() {
      var firstDay = moment(this.dateContext).subtract(
        this.currentDate - 1,
        "days"
      );
      return firstDay.weekday();
    },
    initialDate() {
      return this.today.get("date");
    },
    initialMonth() {
      return this.today.format("MMMM");
    },
    initialYear() {
      return this.today.format("Y");
    },
    numberDaysWeek() {
      let a = this.dateContext.clone().subtract(3, "days").format("YYYY-MM-DD");
      let b = this.dateContext.clone().subtract(2, "days").format("YYYY-MM-DD");
      let c = this.dateContext.clone().subtract(1, "days").format("YYYY-MM-DD");
      let d = this.dateContext.clone().format("YYYY-MM-DD");
      let e = this.dateContext.clone().add(1, "days").format("YYYY-MM-DD");
      let f = this.dateContext.clone().add(2, "days").format("YYYY-MM-DD");
      let g = this.dateContext.clone().add(3, "days").format("YYYY-MM-DD");
      let result = [a, b, c, d, e, f, g];
      return result;
    },
    sanitizedNotes() {
      let result;
      result = validator.trim(this.appointmentNotes);
      result = validator.escape(result);
      return result;
    },
    sanitizedTitle() {
      let result;
      result = validator.trim(this.appointmentDesc);
      result = validator.escape(result);
      return result;
    }
  },
  methods: {
    reset() {
      this.modalCheck = "empty";
      this.appointmentNotes = "";
      this.dateModal2 = moment().format("YYYY-MM-DD");
      this.dateModal = moment().format("YYYY-MM-DD");
      this.appointmentDesc ="";
      this.startHour = "";
      this.endHour = "";
    },
    addMonth() {
      this.dateContext = moment(this.dateContext).add(1, "month");
    },
    subtractMonth() {
      this.dateContext = moment(this.dateContext).subtract(1, "month");
    },
    addWeek() {
      this.dateContext = moment(this.dateContext).add(1, "w");
    },
    subtractWeek() {
      this.dateContext = moment(this.dateContext).subtract(1, "w");
    },
    selectDate(year, month, day) {
      this.dateContext = moment(year + "-" + month + "-" + day);
    },
    JsontoData() {
      let data = localStorage.getItem("testJson");
      let dataArray = JSON.parse(data);
      this.appointmentsList = dataArray;
    },
    appointmentToJson(modalCase) {
      let appointments;
      if(this.appointmentsList) {appointments = this.appointmentsList;}
      else {appointments = [];}
      let appointmentObject = {date: this.dateModal, dateEnd: this.dateModal2, title: this.sanitizedTitle, startHour: this.startHour, endHour: this.endHour, notes: this.sanitizedNotes};
      if(modalCase == 'new') {
        appointments.push(appointmentObject);
      }
      else {
        appointments[modalCase] = appointmentObject;
      }
      let appointmentJson = JSON.stringify(appointments);
      localStorage.setItem("testJson", appointmentJson);
      this.JsontoData();
      this.reset();
    },
    appointmentInDay(date, hour, forModal) {
      let appointmentslist = this.appointmentsList;
      let result = {};
      let modalCheckValue = "new";
      let actualdate = moment(date);
      if(appointmentslist.length > 0) {
         appointmentslist.forEach(function(element, index) {
           if(forModal && actualdate.isBetween(element.date, element.dateEnd, null, '[]')) {
             result = element;
             modalCheckValue = index;
           }
           else {
             if(actualdate.isSame(element.date, 'day') && hour == element.startHour + 1 && element.startHour == 0) {
               result = element;
             }
             if(actualdate.isSame(element.date, 'day') && hour == element.startHour) {
               result = element;
             }
             if(actualdate.isBetween(element.date, element.dateEnd, null, "(]" ) && hour == 1 ) {
               result = element;
             }
             if(actualdate.isBetween(element.date, element.dateEnd)) {
               modalCheckValue = index;
             }
             if(actualdate.isBetween(element.date, element.dateEnd, null, '(]' ) && hour <= element.endHour ) {
               modalCheckValue = index;
             }
             if(actualdate.isSame(element.date, 'day')) {
               if(actualdate.isSame(element.dateEnd, "day")) {
                 if(hour >= element.startHour && hour <= element.endHour) {
                   modalCheckValue = index;
                 }
               }
               else if(hour >= element.startHour) {
                 modalCheckValue = index;
               }
             }
           }
         })
      }
      let returnArray = [result, modalCheckValue];
      return returnArray;
    },
    deleteAppointment(modalCase) {
      let appointments = this.appointmentsList;
      appointments.splice(modalCase, 1);
      this.reset();
      let appointmentJson = JSON.stringify(appointments);
      localStorage.setItem("testJson", appointmentJson);
      this.JsontoData();
    },
    unescape(note) {
      let data = validator.unescape(note);
      return data;
    },
    openModal(param1, param2, param3) {
      let array = this.appointmentInDay(param1, param2, param3);
      this.modalCheck = array[1];
      // {date: this.dateModal, dateEnd: this.dateModal2, title: this.sanitizedTitle, startHour: this.startHour, endHour: this.endHour, notes: this.sanitizedNotes};
      array[0].notes ? this.appointmentNotes = this.unescape(array[0].notes): this.appointmentNotes = "";
      array[0].date ? this.dateModal = array[0].date : this.dateModal = moment().format("YYYY-MM-DD");
      array[0].dateEnd ? this.dateModal2 = array[0].dateEnd : this.dateModal2 = moment().format("YYYY-MM-DD");
      array[0].title ? this.appointmentDesc = unescape(array[0].title) : this.appointmentDesc ="";
      array[0].startHour >= 0 ? this.startHour = array[0].startHour : this.startHour = "";
      array[0].endHour ? this.endHour = array[0].endHour : this.endHour = "";
    },
    getDay(param) {
      return moment(param).date();
    }
  },
  created() {
    this.JsontoData();
  }
};
</script>

<style lang="css">
.margin-0 {
  margin: 0;
}
.padding-note {
  padding-left: 5px;
  padding-right: 5px;
}
.padding-hour {
  padding-left: 16px;
}
.pointer { 
  cursor: pointer; 
}
.default { 
  cursor: default;
}
.weekdays li,
.dates li {
  list-style: none;
  float: left;
  padding: 5px 10px;
  border: 1px solid #eee;
}
.line {
  display: inline;
}
.resize li {
  width: 3vw;
  height: 3vw;
}
@media (max-width: 1250px) {
  .resize li {
    width: 2.75vw;
    height: 2.75vw;
  }
  .CallendarNumber {
    font-size: 9px;
  }
}
@media (max-width: 868px) {
  .resize li {
    width: 2.5vw;
    height: 2.5vw;
  }
  .CallendarNumber {
    font-size: 6px;
  }
}
.leftClear {
  clear: left;
}
.current-day {
  background-color: #ccc;
}
.CallendarNumber {
  font-weight: bold;
  font-size: 14px;
}
div:empty,
.agenda-cell-height {
  height: 3em;
  line-height: 3;
}
#id01, #id02 {
  display: block;
}
.hourHeight {
  height: 7.5vh;
  line-height: 9vh;
}
.agendaHeight {
  height: 5vh;
  line-height: 5vh;
}
.appointPad {
  padding-left: 8px;
}
</style>
