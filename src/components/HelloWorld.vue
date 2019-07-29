<template lang="html">
  <div class="">
<!-- top bar -->
    <div class="w3-cell-row w3-margin-top">
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
    <div class="w3-row">
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
      <div class="w3-rest w3-red">
        <div class="w3-col w3-green default" style="width:9%">
          <div class="w3-row">hours</div>
          <div class="w3-row w3-border" v-for="j in 24" :key="'hour'+j">
            <span v-show="j <=10">0</span>{{j-1}}:00
          </div>
        </div>
        <div v-for="(day, index) in daysAbrev" :key="'days'+index" class="w3-col w3-blue" style="width:13%">
          <div class="w3-row w3-border default">
            <span>{{day}} {{numberDaysWeek[index]}}</span>
          </div>
          <div class="w3-row w3-border pointer" v-for="m in 24" :key="'hour'+m" >
            <div @click="modalCheck = appointmentInDay(numberDaysWeek[index], m -1)[1]">{{appointmentInDay(numberDaysWeek[index], m -1)[0]}}</div>
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
            <input type="text" readonly @click="calendModal = 1" v-bind:placeholder="datevalue"/>
            <select v-model.number="startHour" required>
              <option disabled value="">Select start hour</option>
              <option v-for="hs in 24" :key="'selecthourstart'+hs" v-bind:value="hs-1"><span v-show="hs<=10">0</span>{{hs - 1}}:00</option>
            </select>
            <select v-model.number="endHour">
              <option disabled value="">Select end hour</option>
              <option v-for="he in 24" :key="'selecthourend'+he" v-bind:value="he-1"><span v-show="he<=10">0</span>{{he - 1}}:00</option>
            </select>
            <input type="text" readonly @click="calendModal = 2" v-bind:placeholder="datevalue2"/>
            <div>
              <input type="text" v-model="appointmentDesc" placeholder="add a title" required/>
            </div>
            <div v-if="modalCheck == 'new'">
              <button type="submit" class="w3-btn">Submit</button>
            </div>
            <div v-else>
              <button type="submit" class="w3-btn">Edit</button>
              <button class="w3-btn" @click="deleteAppointment(modalCheck)">Delete</button>
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
export default {
  name: "HelloWorld",
  data() {
    return {
      today: moment(),
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
      var numberUsed = this.dateContext.day();
      let a = this.dateContext.clone().date() - numberUsed;
      let b = this.dateContext.clone().date() - numberUsed + 1;
      let c = this.dateContext.clone().date() - numberUsed + 2;
      let d = this.dateContext.clone().date() - numberUsed + 3;
      let e = this.dateContext.clone().date() - numberUsed + 4;
      let f = this.dateContext.clone().date() - numberUsed + 5;
      let g = this.dateContext.clone().date() - numberUsed + 6;
      let result = [a, b, c, d, e, f, g];
      result.sort();
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
               if(actualdate.isSame(element.date, "day") && hour >= element.startHour && element.endHour >= hour) {
                 result = element.title;
                 modalCheckValue = index;
               }
             }
             else {
               if(actualdate.isSame(element.date, "day") && hour >= element.startHour) {
                 result = element.title;
                 modalCheckValue = index;
               }
               if(actualdate.isBetween(element.date, element.dateEnd)) {
                 result = element.title;
                 modalCheckValue = index;
               }
               if(actualdate.isSame(element.dateEnd, "day") && hour <= element.endHour) {
                 result = element.title;
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
</style>
