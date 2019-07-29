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
      <div class="w3-quarter w3-border">
        <div class="w3-row">
          <div class="w3-half">
            <p>{{month}} {{year}}</p> 
          </div>
          <button type="button" class="w3-btn" name="button" v-on:click="subtractMonth"><i class="fas fa-chevron-left"></i></button>
          <button type="button" class="w3-btn" name="button" v-on:click="addMonth"><i class="fas fa-chevron-right" ></i></button>
        </div>
        <div class="w3-row">
          <ul class="weekdays resize">
            <li class="w3-blue-grey w3-display-container default" :key="index" v-for="(day, index) in days"><div class="w3-display-middle">{{day}}</div></li>
          </ul>
          <ul class="dates resize leftClear">
            <li :key="'blankb' + index + month" v-for="(blank, index) in firstDayOfMonth" class="default">&nbsp;</li>
            <li :key="'dateb' + date + index + month" v-for="(date, index) in daysInMonth" class="w3-display-container pointer" :class="{'current-day': date == initialDate &amp;&amp; month == initialMonth && year == initialYear}" @click="selectDate(year, month, date)" >
              <div class="w3-display-middle CallendarNumber">{{date}}</div>
            </li>
          </ul>
        </div>
      </div>
<!-- /calendar -->
<!-- actual agenda -->
      <div class="w3-rest">
        <div class="w3-col default" style="width:9%">
          <div class="w3-row w3-border w3-center w3-xlarge">{{timezone}}</div>
          <div class="w3-row w3-border w3-center hourHeight" v-for="j in 24" :key="'hour'+j">
            <span v-show="j <=10">0</span>{{j-1}}:00
          </div>
        </div>
        <div v-for="(day, index) in daysAbrev" :key="'days'+index" class="w3-col" style="width:12.5%">
          <div class="w3-row default w3-border w3-center w3-xlarge">
            <span>{{day}} {{numberDaysWeek[index]}}</span>
          </div>
          <div class="w3-row pointer hourHeight appointPad" v-for="m in 24" :key="'hour'+m" @click="modalCheck = appointmentInDay(numberDaysWeek[index], m -1)[1]" :class="{'w3-blue w3-border-left':appointmentInDay(numberDaysWeek[index], m -1)[1] !== 'new', 'w3-border':appointmentInDay(numberDaysWeek[index], m -1)[1] == 'new'}">
              {{appointmentInDay(numberDaysWeek[index], m -1)[0]}}
          </div>
        </div>
      </div>
    </div>
<!-- /actual agenda -->
<!-- modal -->
    <div id="id01" class="w3-modal" v-if="modalCheck !== 'empty'">
      <div class="w3-modal-content">
        <div class="w3-container">
          <span class="w3-button w3-display-topright" @click="modalCheck = 'empty'">&times;</span>
          <form @submit.prevent="appointmentToJson(modalCheck)">
            <div class="w3-row">
              <input type="text" v-model="appointmentDesc" placeholder="Add a title" required class="w3-input w3-center w3-bottombar w3-border-blue w3-section w3-round"/>
            </div>
            <div class="w3-row">
              <input type="text" readonly @click="calendModal = 1" v-bind:placeholder="datevalue" class="w3-input w3-border w3-quarter"/>
              <select v-model.number="startHour" required class="w3-select w3-border w3-quarter">
                <option disabled value="">Select start hour</option>
                <option v-for="hs in 24" :key="'selecthourstart'+hs" v-bind:value="hs-1"><span v-show="hs<=10">0</span>{{hs - 1}}:00</option>
              </select>
              <select v-model.number="endHour" class="w3-select w3-quarter w3-border">
                <option disabled value="">Select end hour</option>
                <option v-for="he in 24" :key="'selecthourend'+he" v-bind:value="he-1"><span v-show="he<=10">0</span>{{he - 1}}:00</option>
              </select>
              <input type="text" readonly @click="calendModal = 2" v-bind:placeholder="datevalue2" class="w3-input w3-border w3-quarter"/>
            </div>
            <div v-if="modalCheck == 'new'" class="w3-section">
              <button type="submit" class="w3-btn w3-blue w3-right w3-margin-bottom">Submit</button>
            </div>
            <div v-else class="w3-section">
              <button class="w3-btn w3-red w3-right w3-margin-bottom" @click="deleteAppointment(modalCheck)"><i class="far fa-trash-alt"></i></button>
              <button type="submit" class="w3-btn w3-blue w3-right w3-margin-bottom">Edit</button>
            </div>
          </form>
        </div>
      </div>
    </div>
<!-- /modal -->
<!-- calendar modal -->
    <div id="id02" class="w3-modal" v-if="calendModal > 0">
      <div class="w3-modal-content">
        <div class="w3-container">
          <div class="w3-row">
        <div class="w3-half">
          <p>{{month}} {{year}}</p> 
        </div>
        <button type="button" class="w3-btn" name="button" v-on:click="subtractMonth"><i class="fas fa-chevron-left"></i></button>
        <button type="button" class="w3-btn" name="button" v-on:click="addMonth"><i class="fas fa-chevron-right" ></i></button>
      </div>
      <div class="w3-row">
        <ul class="weekdays resize">
          <li class="w3-blue-grey w3-display-container" :key="index" v-for="(day, index) in days"><div class="w3-display-middle">{{day}}</div></li>
        </ul>
        <ul class="dates resize leftClear">
          <li :key="'blankb' + index + month" v-for="(blank, index) in firstDayOfMonth">&nbsp;</li>
          <li :key="'dateb' + date + index + month" v-for="(date, index) in daysInMonth" class="w3-display-container" :class="{'current-day': date == initialDate &amp;&amp; month == initialMonth && year == initialYear}" @click="selectDateModal(year, month, date)">
            <div class="w3-display-middle CallendarNumber">{{date}}</div>
          </li>
        </ul>
      </div>
        </div>
      </div>
      
    </div>
<!-- /calendar modal -->
  </div>

</template>

<script>
import moment from "moment";
import momentTimezone from 'moment-timezone';

export default {
  name: "HelloWorld",
  data() {
    return {
      today: moment(),
      timezone: momentTimezone.tz(momentTimezone.tz.guess()).zoneName(),
      dateContext: moment(),
      dateContext2: moment(),
      dateModal: moment(),
      dateModal2: moment(),
      days: ["S", "M", "T", "W", "T", "F", "S"],
      daysAbrev: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"],
      modalCheck: "empty",
      calendModal: 0,
      startHour: "",
      endHour: "",
      appointmentDesc: "",
      appointmentsList: [],
    };
  },
  computed: {
    year() {
      return this.dateContext.format("Y");
    },
    modalyear() {
      return this.dateModal.format("Y");
    },
    modalyear2() {
      return this.dateModal2.format("Y");
    },
    month() {
      return this.dateContext.format("MMMM");
    },
    modalmonth() {
      return this.dateModal.format("MMMM");
    },
    modalmonth2() {
      return this.dateModal2.format("MMMM");
    },
    monthIndex() {
      return this.dateContext.get("month");
    },
    daysInMonth() {
      return this.dateContext.daysInMonth();
    },
    currentDay() {
      return this.dateModal.format("DD");
    },
    currentDay2() {
      return this.dateModal2.format("DD");
    },
    currentDate() {
      return this.dateContext.get("date");
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
      let a = this.dateContext.clone().subtract(3, "days").date();
      let b = this.dateContext.clone().subtract(2, "days").date();
      let c = this.dateContext.clone().subtract(1, "days").date();
      let d = this.dateContext.clone().date();
      let e = this.dateContext.clone().add(1, "days").date();
      let f = this.dateContext.clone().add(2, "days").date();
      let g = this.dateContext.clone().add(3, "days").date();
      let result = [a, b, c, d, e, f, g];
      return result;
    },
    datevalue() {
      return ''+this.modalyear+'-'+this.modalmonth+'-'+this.currentDay
    },
    datevalue2() {
      return ''+this.modalyear2+'-'+this.modalmonth2+'-'+this.currentDay2
    }
  },
  methods: {
    addMonth() {
      this.dateContext = moment(this.dateContext).add(1, "month");
    },
    subtractMonth() {
      this.dateContext = moment(this.dateContext).subtract(1, "month");
    },
    selectDateModal(year, month, day) {
      if(this.calendModal == 1) {
        this.dateModal = moment(year + "-" + month + "-" + day);
        this.dateContext = this.dateModal;
        }
      else if(this.calendModal == 2) {
        this.dateModal2 = moment(year + "-" + month + "-" + day);
        }
      this.calendModal = 0; 
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
      let appointmentObject = {date: this.datevalue, dateEnd: this.datevalue2, title: this.appointmentDesc, startHour: this.startHour, endHour: this.endHour};
      if(modalCase == 'new') {
        appointments.push(appointmentObject);
      }
      else {
        appointments[modalCase] = appointmentObject;
      }
      let appointmentJson = JSON.stringify(appointments);
      localStorage.setItem("testJson", appointmentJson);
      this.startHour = "";
      this.endHour = "";
      this.appointmentDesc = "";
      this.JsontoData();
      this.modalCheck = "empty";
    },
    appointmentInDay(date, hour) {
      let appointmentslist = this.appointmentsList;
      let result = String.fromCharCode(160);
      let modalCheckValue = "new";
      let actualdate = this.dateContext.clone().date(date);
      if(appointmentslist) {
         appointmentslist.forEach(function(element, index) {
           if(element.endHour == "") {
             if(actualdate.isSame(element.date, "day") && hour >= element.startHour) {
               result = element.title;
             }
             else if(actualdate.isSame(element.dateEnd, "day")) {
               result = element.title;
             }
           }
           else {
             if(moment(element.date).isSame(element.dateEnd, "day")) {
               if(actualdate.isSame(element.date, "day") && hour == element.startHour) {
                 result = element.title;
               }
               if(actualdate.isSame(element.date, "day") && hour >= element.startHour && element.endHour >= hour) {
                 modalCheckValue = index;
               }
             }
             else {
               if(actualdate.isSame(element.date, "day") && hour == element.startHour) {
                 result = element.title;
               }
               if(actualdate.isSame(element.date, "day") && hour >= element.startHour) {
                 modalCheckValue = index;
               }
               if(actualdate.isBetween(element.date, element.dateEnd) && hour == 0) {
                 result = element.title;
               }
               if(actualdate.isBetween(element.date, element.dateEnd)) {
                 modalCheckValue = index;
               }
               if(actualdate.isSame(element.dateEnd, "day") && hour == 0) {
                 result = element.title;
               }
               if(actualdate.isSame(element.dateEnd, "day") && hour <= element.endHour) {
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
      this.modalCheck = 'empty';
      let appointmentJson = JSON.stringify(appointments);
      localStorage.setItem("testJson", appointmentJson);
      this.JsontoData();
    }
  },
  created() {
    this.JsontoData();
  }
};
</script>

<style lang="css">
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
  width: 6vw;
  height: 6vw;
}
@media (min-width: 601px) {
  .resize li {
    width: 3vw;
    height: 3vw;
  }
  .CallendarNumber {
    font-size: 9px;
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
  height: 5vh;
  line-height: 5vh;
}
.appointPad {
  padding-left: 8px;
}
</style>
