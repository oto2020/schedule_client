/* eslint-disable */
<template>
  <div class="">

    <div v-show="!fullScreenMode" name="Все кнопки с фильтрами: помещения и переключатели" style="text-align: left;">

      <div class="d-flex flex-wrap">
        <div v-for="section in roomsSections" :key="section" class="m-1">
          <div class="filter-title">{{ section.title }}</div>
          <div class="p-1">
            <div class="list-group">
              <button type="button" v-for="(item, index) in section.rooms" :key="index"
                :class="{ 'room-button list-group-item list-group-item-action active': item.isActive, 'room-button list-group-item list-group-item-action': !item.isActive }"
                @click="handleRoomButtonClick(item)">
                <font-awesome-icon v-if="item.isActive" :icon="['far', 'check-square']" />
                <font-awesome-icon v-if="!item.isActive" :icon="['far', 'square']" />
                {{ item.title }}
              </button>
              <!-- Добавьте другие строки, при необходимости -->
            </div>
          </div>
          <!-- <div class="filter-title">{{ section.title }}</div>
          <button type="button" v-for="(item, index) in section.rooms" :key="index"
            :class="{ 'btn btn-info m-1': item.isActive, 'btn btn-outline-info m-1': !item.isActive }"
            @click="handleRoomButtonClick(item)">
            {{ item.title }}
          </button> -->

        </div>


      </div>


      <div class="d-flex flex-wrap">
        <div name="Выбор коммерческого или бесплатного занятия commercials" class="filter-block m-1">
          <div class="filter-title">По стоимости</div>
          <div class="">
            <button v-for="commercial in commercials" :key="commercial" type="button"
              :class="{ 'filter-button btn active m-1': commercial.isActive, 'filter-button btn m-1': !commercial.isActive }"
              @click="handleCommercialButtonClick(commercial.val)">
              <font-awesome-icon :icon="['fas', commercial.icon]" />
              <br>
              {{ commercial.title }}
            </button>
          </div>
        </div>

        <div name="Выбор режима просмотра полный средний или минимальный modes" class="filter-block m-1">
          <div class="filter-title">Режим просмотра</div>
          <div>
            <button v-for="mode in modes" :key="mode" type="button"
              :class="{ 'filter-button btn active m-1': mode.isActive, 'filter-button btn m-1': !mode.isActive }"
              @click="handleModeButtonClick(mode.val)">
              <font-awesome-icon :icon="['fas', mode.icon]" />
              <br>
              {{ mode.title }}
            </button>
          </div>

        </div>

        <div name="Выбор детских или взрослых занятий kids" class="filter-block m-1">
          <div class="filter-title">Возрастная группа</div>
          <div>
            <button v-for="kid in kids" :key="kid" type="button"
              :class="{ 'filter-button btn active m-1': kid.isActive, 'filter-button btn m-1': !kid.isActive }"
              @click="handleKidButtonClick(kid.val)">
              <font-awesome-icon :icon="['fas', kid.icon]" />
              <br>
              {{ kid.title }}
            </button>
          </div>
        </div>

        <div name="Выбор частей недели weekParts" class="filter-block m-1">
          <div class="filter-title">Выбор дней</div>
          <div>
            <button v-for="wp in weekParts" :key="wp" type="button"
              :class="{ 'filter-button btn active m-1': wp.isActive, 'filter-button btn m-1': !wp.isActive }"
              @click="handleWeekPartButtonClick(wp.val)">
              <font-awesome-icon :icon="['fas', wp.icon]" />
              <br>
              {{ wp.title }}
            </button>
          </div>
        </div>

      </div>


      <div class="m-2 d-inline-block">Отступ справа при печати: <input v-model="paddingRightPercent"
          style="width: 60px; margin-right:4px;">%</div>
      <button @click="generatePDF()">Гнерировать PDF A4 альбом с заданным отступом справа под наши уникальные
        неформальные рамки с отступом {{ paddingRightPercent }}%</button>
    </div>

    <div name="Печатаемый контент" id="printingContent" v-if="fontSizeVW">
      <div id="contentCenter">

        <div class="w-100 fs-4 d-flex pl-4 mt-4">

          <button type="button" class="btn btn-outline-secondary"
            @click="weekNumberDecrementButtonClick"><font-awesome-icon :icon="['fas', 'fa-chevron-left']" /></button>
          <div class="mx-2">{{ computedStartDate }} — {{ computedEndDate }}</div>
          <button type="button" class="btn btn-outline-secondary"
            @click="weekNumberIncrementButtonClick"><font-awesome-icon :icon="['fas', 'fa-chevron-right']" /></button>
          <div name="Перечисление выбранных помещений" class="ml-2" style="margin-left: 10px;">
            {{ this.computedAllowedRoomTitles.join(', ') }}
          </div>

          <div class="ms-auto">

            <!-- <button class="btn me-1" >
            <div v-if="fullScreenMode" class="full-screen-button" @click="toggleFullScreen(false)"> 
              <font-awesome-icon :icon="['fas', 'fa-compress']" />
            </div>
            <div v-else class="full-screen-button"> 
              <font-awesome-icon :icon="['fas', 'fa-expand']" />
            </div>
          </button> -->

            <button class="btn" @click="toggleFullScreen(true)">
              <img v-if="schedule" class="g1logo" :src="require('@/g1logo.png')" alt="Пример изображения">
              <div v-else class="mr-1"><font-awesome-icon icon="spinner" size="lg" spin /></div>
            </button>
          </div>

        </div>

        <!-- Таблица для режима l -->
        <table v-if="computedModeValue === 'l'" class="table table-bordered line-height-1-2 mb-4">
          <thead>
            <tr>
              <th class="col-md-1">Часы</th>
              <th v-for="(day, index) in computedActiveWeekDays" :key="index" class="col-md-1">{{ day }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(hourData, hour) in scheduleData" :key="hour">
              <td>{{ hour }}</td>
              <td v-for="(dayData, day) in hourData" :key="day" class="td-container">
                <div v-if="dayData" class="inner-div">
                  <div v-for="l in dayData" :key="l">
                    <div class="lesson-parent m-1" :style="{ border: '1px solid ' + l.backgroundColor }">
                      <div>
                        <div class="exercise-title">{{ l.exerciseTitle }}</div>
                        <div class="trainer-name text-truncate d-inline-block">{{ l.trainerName }}</div>
                        <div class="start-end-time">{{ l.startTime }} - {{ l.endTime }}</div>
                        <div class="room-title">{{ l.roomTitle }}</div>
                        <div class="room-title">{{ l.dayOfWeek }}</div>
                      </div>
                      <span class="lesson-color" :style="{ background: l.backgroundColor }"></span>
                      <div v-if="l.exerciseTitle.includes('₽')" class="ruble-icon">
                        <font-awesome-icon :icon="['fas', 'ruble-sign']" size="sm" />
                      </div>
                    </div>
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>

        <!-- Таблица для режима m -->
        <table v-if="computedModeValue === 'm'" class="table table-bordered line-height-1-2 mb-4">
          <thead>
            <tr>
              <th class="col-md-1">Часы</th>
              <th v-for="(day, index) in computedActiveWeekDays" :key="index" class="col-md-1">{{ day }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(hourData, hour) in scheduleData" :key="hour">
              <td>{{ hour }}</td>
              <td v-for="(dayData, day) in hourData" :key="day" class="td-container">
                <div v-if="dayData" class="inner-div">
                  <div v-for="l in dayData" :key="l">
                    <div class="lesson-parent m-1" :style="{ border: '1px solid ' + l.backgroundColor }">
                      <div>
                        <div class="exercise-title">{{ l.exerciseTitle }}</div>
                        <div class="trainer-name text-truncate d-inline-block">{{ l.trainerName }}</div>
                        <div class="start-end-time">{{ l.startTime }} - {{ l.endTime }}</div>
                        <div class="room-title">{{ l.roomTitle }}</div>
                        <div class="room-title">{{ l.dayOfWeek }}</div>
                      </div>
                      <span class="lesson-color" :style="{ background: l.backgroundColor }"></span>
                      <div v-if="l.exerciseTitle.includes('₽')" class="ruble-icon">
                        <font-awesome-icon :icon="['fas', 'ruble-sign']" size="sm" />
                      </div>
                    </div>
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>

        <!-- Таблица для режима s -->
        <table v-if="computedModeValue === 's'" class="table table-bordered line-height-1-2 mb-4">
          <thead>
            <tr>
              <th class="col-md-1">Часы</th>
              <th v-for="(day, index) in computedActiveWeekDays" :key="index" class="col-md-1">{{ day }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(hourData, hour) in scheduleData" :key="hour">
              <td v-for="(dayData, day) in hourData" :key="day" class="td-container">
                <div v-if="dayData" class="inner-div">
                  <div v-for="l in dayData" :key="l">
                    <div class="lesson-parent m-1" :style="{ border: '1px solid ' + l.backgroundColor }">
                      <div>
                        <div class="exercise-title">{{ l.exerciseTitle }}</div>
                        <div class="trainer-name text-truncate d-inline-block">{{ l.trainerName }}</div>
                        <div class="start-end-time">{{ l.startTime }} - {{ l.endTime }}</div>
                        <div class="room-title">{{ l.roomTitle }}</div>
                        <div class="room-title">{{ l.dayOfWeek }}</div>
                      </div>
                      <span class="lesson-color" :style="{ background: l.backgroundColor }"></span>
                      <div v-if="l.exerciseTitle.includes('₽')" class="ruble-icon">
                        <font-awesome-icon :icon="['fas', 'ruble-sign']" size="sm" />
                      </div>
                    </div>
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>





      </div>
      <div id="contentRight"></div>
    </div>




  </div>
</template>

<script>
/* eslint-disable no-mixed-spaces-and-tabs */
import axios from 'axios';
import { reactive } from 'vue';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

export default {
  name: 'MainComponent',

  data() {
    return {
      fontSizeVW: '100%', // размер текста в зависимости от ширины страницы
      paddingRightPercent: 6.4, // ширина отступа справа
      fullScreenMode: false,
      year: 0, // 2024,
      weekNumber: 0, // 2,
      startDate: '', // "2024-01-08T00:00:00"
      endDate: '',   // "2024-01-14T23:59:59"
      kids: [
        { title: 'Любые', icon: 'users', val: 'all', isActive: false },
        { title: 'Детские', icon: 'child', val: 'kid', isActive: false },
        { title: 'Взрослые', icon: 'user-tie', val: 'non-kid', isActive: true },
      ],
      commercials: [
        { title: 'Неважно', val: 'all', isActive: true, icon: 'question' },
        { title: 'Платные', val: 'commercial', isActive: false, icon: 'ruble-sign' },
        { title: 'Бесплатные', val: 'non-commercial', isActive: false, icon: 'fa-gift' },
      ],
      modes: [
        { title: 'Полный', val: 'l', isActive: true, icon: 'expand' },
        { title: 'Средний', val: 'm', isActive: false, icon: 'compress' },
        { title: 'Минимум', val: 's', isActive: false, icon: 'compress-arrows-alt' },
      ],

      weekParts: [
        { title: 'Пн, Вт', icon: 'calendar', val: '0,1', isActive: false },
        { title: 'Ср, Чт', icon: 'calendar', val: '2,3', isActive: false },
        { title: 'Пт, Сб, Вс', icon: 'calendar', val: '4,5,6', isActive: false },
        { title: 'Вся неделя', icon: 'calendar', val: '0,1,2,3,4,5,6', isActive: true },
      ],
      schedule: [],
      rooms: [],
      roomsSections: [
        {
          "title": "ТЗ и студии",
          "icon": "tz",
          "rooms": [
            { "isActive": false, "title": "Тренажерный зал" },
            { "isActive": false, "title": "Студия пилатес" },
            { "isActive": false, "title": "Студия йоги" },
          ]
        },
        {
          "title": "Кроссфит и единоборства",
          "icon": "ring",
          "rooms": [
            { "isActive": false, "title": "Зона кроссфита" },
            { "isActive": false, "title": "Ринг" }
          ]
        },
        {
          "title": "Групповые программы",
          "icon": "gp",
          "rooms": [
            { "isActive": true, "title": "Зал №1" },
            { "isActive": true, "title": "Зал №2" }
          ]
        },
        {
          "title": "Сайкл",
          "icon": "cycle",
          "rooms": [
            { "isActive": false, "title": "Сайкл студия" },
          ]
        },
        {
          "title": "Бассейны",
          "icon": "bassei",
          "rooms": [
            { "isActive": false, "title": "Аква зона" },
            { "isActive": false, "title": "Малый бассейн" },
          ]
        },
        {
          "title": "Спорт (дети)",
          "icon": "gun1or-sport",
          "rooms": [
            { "isActive": false, "title": "Зал единоборств KIDS" },
            { "isActive": false, "title": "Эстетический зал KIDS" }
          ]
        },
        {
          "title": "Интеллект и творчество (дети)",
          "icon": "gun1or-intellect",
          "rooms": [
            { "isActive": false, "title": "Студия ИНТЕЛЛЕКТ KIDS" },
            { "isActive": false, "title": "Студия КРЕАТИВ KIDS" },
          ]
        },
      ],
      scheduleData: reactive({}),
      testMode: false,
      scheduleDataTest: { "7": null, "8": null, "9": { "понедельник": [], "вторник": [{ "exerciseTitle": "ХАТХА YOGA 60", "trainerName": "Шакура  Светлана", "startTime": "09:00", "endTime": "10:00", "hour": 9, "dayOfWeek": "вторник", "dateOfMonth": 9, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#B8C6EA" }], "среда": [], "четверг": [{ "exerciseTitle": "ХАТХА YOGA 60", "trainerName": "Шакура  Светлана", "startTime": "09:00", "endTime": "10:00", "hour": 9, "dayOfWeek": "четверг", "dateOfMonth": 11, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#B8C6EA" }], "пятница": [], "суббота": [{ "exerciseTitle": "СУСТАВНАЯ ГИМНАСТИКА", "trainerName": "Замостьянин Валерий", "startTime": "09:00", "endTime": "10:00", "hour": 9, "dayOfWeek": "суббота", "dateOfMonth": 13, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "воскресенье": [] }, "10": { "понедельник": [], "вторник": [{ "exerciseTitle": "STRETCHING", "trainerName": "Рогозинская Елена", "startTime": "10:00", "endTime": "11:00", "hour": 10, "dayOfWeek": "вторник", "dateOfMonth": 9, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#1974CE" }], "среда": [{ "exerciseTitle": "PILATES", "trainerName": "Волкова  Виктория ", "startTime": "10:00", "endTime": "11:00", "hour": 10, "dayOfWeek": "среда", "dateOfMonth": 10, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "четверг": [{ "exerciseTitle": "МЕДИТАЦИЯ В ГАМАКАХ ₽*", "trainerName": "Шакура  Светлана", "startTime": "10:00", "endTime": "11:00", "hour": 10, "dayOfWeek": "четверг", "dateOfMonth": 11, "exerciseDuration": 60, "roomTitle": "Студия йоги ", "backgroundColor": "#FFD700" }, { "exerciseTitle": "STRETCHING", "trainerName": "Рогозинская Елена", "startTime": "10:00", "endTime": "11:00", "hour": 10, "dayOfWeek": "четверг", "dateOfMonth": 11, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#1974CE" }], "пятница": [{ "exerciseTitle": "ZUMBA", "trainerName": "Николаенко Анна", "startTime": "10:00", "endTime": "11:00", "hour": 10, "dayOfWeek": "пятница", "dateOfMonth": 12, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#EE82EE" }], "суббота": [{ "exerciseTitle": "ХАТХА YOGA 60", "trainerName": "Шакура  Светлана", "startTime": "10:00", "endTime": "11:00", "hour": 10, "dayOfWeek": "суббота", "dateOfMonth": 13, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#B8C6EA" }], "воскресенье": [] }, "11": { "понедельник": [{ "exerciseTitle": "STRETCHING", "trainerName": "Волкова  Виктория ", "startTime": "11:00", "endTime": "12:00", "hour": 11, "dayOfWeek": "понедельник", "dateOfMonth": 8, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#1974CE" }], "вторник": [{ "exerciseTitle": "UNIFLY ₽", "trainerName": "Заремблюк Наталия", "startTime": "11:00", "endTime": "11:55", "hour": 11, "dayOfWeek": "вторник", "dateOfMonth": 9, "exerciseDuration": 55, "roomTitle": "Студия йоги ", "backgroundColor": "#FFD700" }], "среда": [{ "exerciseTitle": "YOGA HEALTH ₽", "trainerName": "Рогозинская Елена", "startTime": "11:00", "endTime": "12:20", "hour": 11, "dayOfWeek": "среда", "dateOfMonth": 10, "exerciseDuration": 80, "roomTitle": "Студия йоги ", "backgroundColor": "#A6CAF0" }, { "exerciseTitle": "TOTAL MAX & BOSU/FITBALL", "trainerName": "Сысоева Олеся", "startTime": "11:00", "endTime": "11:55", "hour": 11, "dayOfWeek": "среда", "dateOfMonth": 10, "exerciseDuration": 55, "roomTitle": "Зал №1", "backgroundColor": "#FBED9E" }], "четверг": [{ "exerciseTitle": "СУСТАВНАЯ ГИМНАСТИКА", "trainerName": "Замостьянин Валерий", "startTime": "11:00", "endTime": "12:00", "hour": 11, "dayOfWeek": "четверг", "dateOfMonth": 11, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "пятница": [{ "exerciseTitle": "UNIFLY ₽", "trainerName": "Заремблюк Наталия", "startTime": "11:00", "endTime": "11:55", "hour": 11, "dayOfWeek": "пятница", "dateOfMonth": 12, "exerciseDuration": 55, "roomTitle": "Студия йоги ", "backgroundColor": "#FFD700" }, { "exerciseTitle": "STRONG NATION", "trainerName": "Николаенко Анна", "startTime": "11:00", "endTime": "12:00", "hour": 11, "dayOfWeek": "пятница", "dateOfMonth": 12, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#D02090" }], "суббота": [], "воскресенье": [{ "exerciseTitle": "PILATES", "trainerName": "Сысоева Олеся", "startTime": "11:00", "endTime": "12:00", "hour": 11, "dayOfWeek": "воскресенье", "dateOfMonth": 14, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }] }, "12": { "понедельник": [{ "exerciseTitle": "UNIFLY ₽", "trainerName": "Заремблюк Наталия", "startTime": "12:00", "endTime": "12:55", "hour": 12, "dayOfWeek": "понедельник", "dateOfMonth": 8, "exerciseDuration": 55, "roomTitle": "Студия йоги ", "backgroundColor": "#FFD700" }, { "exerciseTitle": "ANIMAL FLOW ₽*", "trainerName": "Волкова  Виктория ", "startTime": "12:00", "endTime": "13:00", "hour": 12, "dayOfWeek": "понедельник", "dateOfMonth": 8, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#FFB6C1" }], "вторник": [], "среда": [], "четверг": [{ "exerciseTitle": "PILATES", "trainerName": "Волкова  Виктория ", "startTime": "12:00", "endTime": "13:00", "hour": 12, "dayOfWeek": "четверг", "dateOfMonth": 11, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "пятница": [], "суббота": [], "воскресенье": [] }, "13": null, "14": { "понедельник": [], "вторник": [{ "exerciseTitle": "СУСТАВНАЯ ГИМНАСТИКА", "trainerName": "Замостьянин Валерий", "startTime": "14:00", "endTime": "15:00", "hour": 14, "dayOfWeek": "вторник", "dateOfMonth": 9, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "среда": [], "четверг": [], "пятница": [], "суббота": [], "воскресенье": [] }, "15": { "понедельник": [], "вторник": [{ "exerciseTitle": "PILATES", "trainerName": "Сысоева Олеся", "startTime": "15:00", "endTime": "16:00", "hour": 15, "dayOfWeek": "вторник", "dateOfMonth": 9, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "среда": [{ "exerciseTitle": "ZUMBA GOLD", "trainerName": "Николаенко Анна", "startTime": "15:00", "endTime": "16:00", "hour": 15, "dayOfWeek": "среда", "dateOfMonth": 10, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#EE82EE" }], "четверг": [{ "exerciseTitle": "CIRCL MOBILITY", "trainerName": "Николаенко Анна", "startTime": "15:00", "endTime": "16:00", "hour": 15, "dayOfWeek": "четверг", "dateOfMonth": 11, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "пятница": [{ "exerciseTitle": "СУСТАВНАЯ ГИМНАСТИКА", "trainerName": "Замостьянин Валерий", "startTime": "15:00", "endTime": "16:00", "hour": 15, "dayOfWeek": "пятница", "dateOfMonth": 12, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "суббота": [], "воскресенье": [] }, "16": null, "17": null, "18": { "понедельник": [], "вторник": [{ "exerciseTitle": "STRONG NATION", "trainerName": "Николаенко Анна", "startTime": "18:00", "endTime": "19:00", "hour": 18, "dayOfWeek": "вторник", "dateOfMonth": 9, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#D02090" }], "среда": [], "четверг": [{ "exerciseTitle": "UNIFLY ₽", "trainerName": "Заремблюк Наталия", "startTime": "18:00", "endTime": "18:55", "hour": 18, "dayOfWeek": "четверг", "dateOfMonth": 11, "exerciseDuration": 55, "roomTitle": "Студия йоги ", "backgroundColor": "#FFD700" }, { "exerciseTitle": "ANIMAL FLOW ₽*", "trainerName": "Николаенко Анна", "startTime": "18:00", "endTime": "19:00", "hour": 18, "dayOfWeek": "четверг", "dateOfMonth": 11, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#FFB6C1" }], "пятница": [{ "exerciseTitle": "ART FUNCTIONAL", "trainerName": "Волкова  Виктория ", "startTime": "18:00", "endTime": "18:55", "hour": 18, "dayOfWeek": "пятница", "dateOfMonth": 12, "exerciseDuration": 55, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "суббота": [], "воскресенье": [] }, "19": { "понедельник": [], "вторник": [{ "exerciseTitle": "ZUMBA", "trainerName": "Николаенко Анна", "startTime": "19:00", "endTime": "20:00", "hour": 19, "dayOfWeek": "вторник", "dateOfMonth": 9, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#EE82EE" }], "среда": [{ "exerciseTitle": "UNIFLY ₽", "trainerName": "Заремблюк Наталия", "startTime": "19:00", "endTime": "19:55", "hour": 19, "dayOfWeek": "среда", "dateOfMonth": 10, "exerciseDuration": 55, "roomTitle": "Студия йоги ", "backgroundColor": "#FFD700" }, { "exerciseTitle": "PROPILATES ₽", "trainerName": "Волкова  Виктория ", "startTime": "19:00", "endTime": "19:55", "hour": 19, "dayOfWeek": "среда", "dateOfMonth": 10, "exerciseDuration": 55, "roomTitle": "Зал №1", "backgroundColor": null }], "четверг": [{ "exerciseTitle": "ZUMBA", "trainerName": "Николаенко Анна", "startTime": "19:00", "endTime": "20:00", "hour": 19, "dayOfWeek": "четверг", "dateOfMonth": 11, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#EE82EE" }], "пятница": [{ "exerciseTitle": "STRETCHING", "trainerName": "Волкова  Виктория ", "startTime": "19:00", "endTime": "20:00", "hour": 19, "dayOfWeek": "пятница", "dateOfMonth": 12, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#1974CE" }], "суббота": [], "воскресенье": [] }, "20": { "понедельник": [], "вторник": [{ "exerciseTitle": "CIRCL MOBILITY", "trainerName": "Николаенко Анна", "startTime": "20:00", "endTime": "21:00", "hour": 20, "dayOfWeek": "вторник", "dateOfMonth": 9, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#CCFFCC" }], "среда": [{ "exerciseTitle": "ХАТХА YOGA 60", "trainerName": "Шакура  Светлана", "startTime": "20:00", "endTime": "21:00", "hour": 20, "dayOfWeek": "среда", "dateOfMonth": 10, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#B8C6EA" }], "четверг": [], "пятница": [{ "exerciseTitle": "ХАТХА YOGA 60", "trainerName": "Шакура  Светлана", "startTime": "20:00", "endTime": "21:00", "hour": 20, "dayOfWeek": "пятница", "dateOfMonth": 12, "exerciseDuration": 60, "roomTitle": "Зал №1", "backgroundColor": "#B8C6EA" }], "суббота": [], "воскресенье": [] }, "21": null, "22": null },
      weekDays: ['понедельник', 'вторник', 'среда', 'четверг', 'пятница', 'суббота', 'воскресенье'],
    };
  },

  mounted() {
    // // Считываем значение из LocalStorage при запуске
    // const storedValue = localStorage.getItem('myVariable');
    // if (storedValue) {
    //   this.myVariable = storedValue;
    // }
    // // Сохраняем новое значение в LocalStorage при модификации
    // localStorage.setItem('myVariable', this.myVariable);
    this.year = this.getCurrentYear();
    this.weekNumber = this.getCurrentWeekNumber();
    this.setWeekStartEndDate(this.year, this.weekNumber);
    // Код, который должен выполниться после загрузки компонента
    console.log('Компонент загружен!');
    if (this.testMode == false) {
      this.fetchData();
    }
  },

  // Возвращает вычисленное значение на основе данных
  computed: {
    computedAllowedRoomTitles() {
      let allowedRoomTitles = [];
      this.roomsSections.forEach((curr) => { allowedRoomTitles.push(...curr.rooms.filter(item => item.isActive).map(el => el.title)); });
      return allowedRoomTitles;
    },
    computedStartDate() {
      const options = { day: 'numeric', month: 'long' };
      const date = new Date(this.startDate);
      return date.toLocaleDateString('ru-RU', options);
    },
    computedEndDate() {
      const options = { day: 'numeric', month: 'long' };
      const date = new Date(this.endDate);
      return date.toLocaleDateString('ru-RU', options);
    },
    // кнопка переключения детских и взрослых направлений
    computedKidValue() {
      return this.kids.find((kid) => kid.isActive).val;
    },
    // кнопка переключения режимов работы
    computedModeValue() {
      return this.modes.find((mode) => mode.isActive).val;
    },
    // кнопка переключения режимов платные/бесплатные/неважно
    computedCommercialValue() {
      return this.commercials.find((commercial) => commercial.isActive).val;
    },
    // кнопка переключения частей недели
    computedWeekPartValue() {
      return this.weekParts.find((wp) => wp.isActive).val;
    },
    computedActiveWeekDays() {
      // Получим список уникальных номеров дней недели, у которых есть занятия
      const activeDays = new Set();

      Object.values(this.scheduleData).forEach(hourRow => {
        if (hourRow && typeof hourRow === 'object') {
          Object.entries(hourRow).forEach(([, lessons]) => {
            if (lessons && lessons.length > 0) {
              lessons.forEach(lesson => {
                if (lesson.dayOfWeekNumber !== undefined) {
                  activeDays.add(lesson.dayOfWeekNumber);
                }
              });
            }
          });
        }
      });

      // Преобразуем номера дней обратно в имена из weekDays
      return this.weekDays.filter((_, index) => activeDays.has(index));
    }
  },
  methods: {
    toggleFullScreen() {
      this.fullScreenMode = !this.fullScreenMode;
    },
    weekNumberIncrementButtonClick() {
      if (this.weekNumber == 52) {
        this.weekNumber = 1;
        this.year++;
      }
      else {
        this.weekNumber++;
      }
      this.setWeekStartEndDate(this.year, this.weekNumber);
      this.fetchData();
    },
    weekNumberDecrementButtonClick() {
      if (this.weekNumber == 1) {
        this.weekNumber = 52;
        this.year--;
      }
      else {
        this.weekNumber--;
      }
      this.setWeekStartEndDate(this.year, this.weekNumber);
      this.fetchData();
    },
    setWeekStartEndDate(year, weekNumber) {
      const januaryFirst = new Date(year, 0, 1);
      const daysToMonday = (1 - januaryFirst.getDay() + 7) % 7; // дней до понедельника
      const daysToTargetWeek = (weekNumber - 1) * 7;

      const currentWeekStartDate = new Date(januaryFirst);
      currentWeekStartDate.setDate(januaryFirst.getDate() + daysToMonday + daysToTargetWeek);
      currentWeekStartDate.setHours(0, 0, 0, 0);

      const nextWeekStartDate = new Date(currentWeekStartDate);
      nextWeekStartDate.setDate(currentWeekStartDate.getDate() + 7);

      const currentWeekEndDate = new Date(nextWeekStartDate);
      currentWeekEndDate.setSeconds(currentWeekEndDate.getSeconds() - 1);

      console.log(currentWeekStartDate);
      console.log(nextWeekStartDate);
      this.startDate = this.formatDateToISOStringWithoutMilliseconds(currentWeekStartDate);
      this.endDate = this.formatDateToISOStringWithoutMilliseconds(currentWeekEndDate);
    },
    // Вспомогательная функция
    formatDateToISOStringWithoutMilliseconds(date) {
      const year = date.getFullYear();
      const month = (date.getMonth() + 1).toString().padStart(2, '0');
      const day = date.getDate().toString().padStart(2, '0');
      const hours = date.getHours().toString().padStart(2, '0');
      const minutes = date.getMinutes().toString().padStart(2, '0');
      const seconds = date.getSeconds().toString().padStart(2, '0');

      return `${year}-${month}-${day}T${hours}:${minutes}:${seconds}`;
    },
    getMondayMidnightDate(weekNumber, year) {
      const januaryFirst = new Date(year, 0, 1);
      const daysToMonday = (1 - januaryFirst.getDay() + 7) % 7; // дней до понедельника
      const daysToTargetWeek = (weekNumber - 1) * 7;

      const targetDate = new Date(januaryFirst);
      targetDate.setDate(januaryFirst.getDate() + daysToMonday + daysToTargetWeek);

      // Устанавливаем время полуночи
      targetDate.setHours(0, 0, 0, 0);

      return targetDate;
    },
    getCurrentYear() {
      const currentDate = new Date();
      const currentYear = currentDate.getFullYear();
      return currentYear;
    },
    getCurrentWeekNumber() {
      const currentDate = new Date();
      currentDate.setHours(0, 0, 0, 0);

      // Устанавливаем 4 января как первый день недели (ISO 8601)
      currentDate.setDate(currentDate.getDate() + 4 - (currentDate.getDay() || 7));

      // Получаем номер недели
      const yearStart = new Date(currentDate.getFullYear(), 0, 1);
      const weekNumber = Math.ceil(((currentDate - yearStart) / 86400000 + 1) / 7);

      return weekNumber;
    },


    // Применяет настройки после изменений фильтров
    applySettings() {

      let filteredSchedule = this.schedule.filter(el => this.computedAllowedRoomTitles.includes(el.roomTitle));
      if (this.computedCommercialValue === 'commercial') {
        filteredSchedule = filteredSchedule.filter(el => el.exerciseTitle.includes('₽'));
      }
      if (this.computedCommercialValue === 'non-commercial') {
        filteredSchedule = filteredSchedule.filter(el => !el.exerciseTitle.includes('₽'));
      }

      if (this.computedKidValue === 'kid') {
        filteredSchedule = filteredSchedule.filter(el => el.exerciseTitle.includes('KIDS'));
      }
      if (this.computedKidValue === 'non-kid') {
        filteredSchedule = filteredSchedule.filter(el => !el.exerciseTitle.includes('KIDS'));
      }

      // Фильтрация по дням недели
      if (this.computedWeekPartValue) {
        const allowedDays = this.computedWeekPartValue
          .split(',')
          .map(str => str.trim())
          .filter(str => str !== '')
          .map(Number); // получаем массив чисел, например [0, 1, 2]

        filteredSchedule = filteredSchedule.filter(el => allowedDays.includes(Number(el.dayOfWeekNumber)));
      }



      // обнуляем данные таблицы
      this.scheduleData = [];
      if (this.computedModeValue === 'l') {
        let result = {};

        // Парсим допустимые дни недели из computedWeekPartValue
        const allowedDayNumbers = this.computedWeekPartValue
          .split(',')
          .map(str => str.trim())
          .filter(str => str !== '')
          .map(Number); // например: [0, 1, 2]

        const allowedDays = allowedDayNumbers.map(num => this.weekDays[num]);

        // Определяем, какие из этих дней реально используются
        const usedDays = new Set(
          filteredSchedule
            .map(el => el.dayOfWeek)
            .filter(day => allowedDays.includes(day))
        );

        // Инициализируем строки от 7 до 22
        for (let hour = 7; hour <= 22; hour++) {
          result[hour] = {};
          usedDays.forEach(day => {
            const entries = filteredSchedule.filter(
              item => item.hour === hour && item.dayOfWeek === day
            );
            result[hour][day] = entries.length > 0 ? entries : null;
          });
        }

        this.scheduleData = result;
      }


      if (this.computedModeValue === 'm') {
        let result = {};

        filteredSchedule.forEach(item => {
          const hour = item.hour;
          const day = item.dayOfWeek;

          if (!result[hour]) {
            result[hour] = {};
          }

          if (!result[hour][day]) {
            result[hour][day] = [];
          }

          result[hour][day].push(item);
        });

        this.scheduleData = result;
      }



      // режим просмотра: самый компактный small
      if (this.computedModeValue === 's') {

        filteredSchedule = filteredSchedule
          .sort((a, b) => a.hour - b.hour)
          .sort((a, b) => a.exerciseTitle - b.exerciseTitle);
        let result = {};

        // у каждого дня недели свой номер столбца
        let dayOfWeekArray = this.weekDays.map(el => { return { title: el, row: 0 }; });
        console.log(dayOfWeekArray);
        // пробегаемся по всем элементам расписания и закидываем их в соответствующую ячейку, после чего column++
        filteredSchedule.forEach(exercise => {
          // День недели из преобразованного массива dayOfWeekArray в соответствии с перебираемым занятием
          let currentDayOfWeekElem = dayOfWeekArray.find(el => el.title === exercise.dayOfWeek);
          let currentDayOfWeekElemIndex = dayOfWeekArray.findIndex(el => el.title === exercise.dayOfWeek);
          if (!result[currentDayOfWeekElem.row]) {
            result[currentDayOfWeekElem.row] = [];
          }
          if (!result[currentDayOfWeekElem.row][currentDayOfWeekElemIndex]) {
            result[currentDayOfWeekElem.row][currentDayOfWeekElemIndex] = [];
          }
          result[currentDayOfWeekElem.row][currentDayOfWeekElemIndex].push(exercise);
          console.log(`${currentDayOfWeekElem.row}, ${currentDayOfWeekElemIndex}: ${exercise}`)
          currentDayOfWeekElem.row++;
        })
        this.scheduleData = result;
      }




      // // Создаем объект для хранения массивов по дням недели
      // ['понедельник', 'вторник', 'среда', 'четверг', 'пятница', 'суббота', 'воскресенье'].forEach(day => {
      //     this.scheduleData[day] = filteredSchedule
      //         .filter(el => el.dayOfWeek === day)
      //         .sort((a, b) => a.hour - b.hour)
      //         .sort((a, b) => a.exerciseTitle - b.exerciseTitle);
      // });

      // console.log(this.scheduleData);

    },


    // изменения набора выбранных помещений
    handleRoomButtonClick(item) {
      item.isActive = !item.isActive;
      this.applySettings();
    },

    // изменение режима просмотра
    handleModeButtonClick(modeVal) {
      this.modes.forEach(m => m.isActive = m.val == modeVal);
      this.applySettings();
    },

    // изменение выбора платные/бесплатные занятия
    handleCommercialButtonClick(commercialVal) {
      this.commercials.forEach(c => c.isActive = c.val == commercialVal);
      this.applySettings();
    },

    // изменение выбора взрослые/детские направления
    handleKidButtonClick(kidVal) {
      this.kids.forEach(k => k.isActive = k.val == kidVal);
      this.applySettings();
    },

    // изменение выбора части недели
    handleWeekPartButtonClick(wpVal) {
      this.weekParts.forEach(wp => wp.isActive = wp.val == wpVal);
      this.applySettings();
    },

    // генерирование ПДФ-файла
    async generatePDF() {
      // this.fontSizeVW = '2rem';
      // this.$forceUpdate();
      const printingContent = document.getElementById('printingContent');
      const contentCenter = document.getElementById('contentCenter');
      const contentRight = document.getElementById('contentRight');
      const oldWidthPrintingContent = printingContent.offsetWidth;
      // Константы для размеров листа A4 в альбомной ориентации
      const A4_WIDTH = 297;
      const A4_HEIGHT = 210;
      // Вычисляем соотношение сторон листа A4
      const a4AspectRatio = A4_WIDTH / A4_HEIGHT;
      // Новая ширина


      const newWidthPrintingContent = printingContent.offsetHeight * a4AspectRatio;
      // Устанавливаем новые размеры div
      if (newWidthPrintingContent > oldWidthPrintingContent)
        printingContent.style.width = newWidthPrintingContent + 'px';

      // устанавливаем новые размеры для того, чтобы справа сформировался отступ:
      let contentCenterWidth = Math.floor(printingContent.offsetWidth * (100 - this.paddingRightPercent) / 100);
      let contentRightWidth = Math.ceil(printingContent.offsetWidth * (this.paddingRightPercent) / 100);

      contentCenter.style.width = contentCenterWidth + 'px';
      contentRight.style.width = contentRightWidth + 'px';

      // Use html2canvas to capture the HTML printingContent
      const canvas = await html2canvas(printingContent);
      // Create a new jsPDF instance
      const pdf = new jsPDF({
        orientation: "landscape",
        unit: "px",
        format: [printingContent.offsetWidth, printingContent.offsetHeight]
      });
      // Add the captured image to the PDF
      pdf.addImage(canvas.toDataURL('image/jpeg'), 'JPEG', 0, 0, printingContent.offsetWidth, printingContent.offsetHeight);
      console.log(printingContent.offsetWidth, printingContent.offsetHeight);
      // Save or display the PDF
      let fileName = `${this.computedStartDate} — ${this.computedEndDate}, ${this.computedAllowedRoomTitles.join(',')}`
      pdf.save(fileName + '.pdf');

      // возвращаем прежнюю ширину:
      printingContent.style.width = oldWidthPrintingContent + 'px';

      contentCenter.style.width = oldWidthPrintingContent + 'px';
      contentRight.style.width = '0 px';

      // this.fontSizeVW = '1rem';
    },

    // получает несгруппированный массив расписания и записывает его в this.schedule
    async fetchData() {
      try {
        this.schedule = null;
        console.log(process.env.VUE_APP_API_URL);
        const response = await axios.get(`${process.env.VUE_APP_API_URL}/schedule/current?startDate=${this.startDate}&endDate=${this.endDate}`);
        this.schedule = response.data.data;
        // заполняем набор комнат
        this.rooms = [...new Set(this.schedule.map(item => item.roomTitle))].sort().map(el => { return { isActive: false, title: el } });
        // вызываем отрисовку расписания
        this.applySettings();
        console.log(`Обновлено ${response.data.fetchedAtHuman}`);
      } catch (error) {
        console.error('Ошибка при получении данных:', error);
      }

    }

  },

  props: {
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#contentCenter {
  padding: 2px;
  border-right: 1px dashed rgb(131, 130, 130);
}

.full-screen-button {
  -webkit-transition: all 0.3s ease;
  -o-transition: all 0.3s ease;
  transition: all 0.3s ease;
  height: 27px;
  width: 27px;
}

.g1logo {
  -webkit-transition: all 0.3s ease;
  -o-transition: all 0.3s ease;
  transition: all 0.3s ease;
  height: 27px;
  width: 100px;
}

.room-button {
  padding: 5px;
  font-size: 11pt;
}

button {

  border-radius: 0;
  font-weight: bold;
  border-color: #f33;
  color: #333;
  /* Цвет текста для активного состояния */
  border: 1px solid #f33;
}

button.active,
button:hover {
  font-weight: bold;
  transition: background-color 0.3s ease;
  /* Плавное изменение цвета за 0.3 секунды с эффектом ease-in-out */
  background-color: #f33;
  /* Замените 'your_custom_color' на свой цвет */
  border-color: #333;
  color: #fff;
  /* Цвет текста для активного состояния */
}

.list-group {

  border-radius: 0;
}

.list-group-item:hover,
.list-group-item.active {
  font-weight: bold;
  transition: background-color 0.3s ease;
  /* Плавное изменение цвета за 0.3 секунды с эффектом ease-in-out */
  background-color: #f33;
  /* Замените 'your_custom_color' на свой цвет */
  border-color: #333;
  color: #fff;
  /* Цвет текста для активного состояния */
}


.filter-block {
  /* min-width: 430px; */
  padding: 0px;
}

.filter-title {
  padding-left: 5px;
  font-size: 10pt;
}

.filter-button {
  text-align: center;
  font-size: 12pt;
}

.ruble-icon {
  position: absolute;
  bottom: 3px;
  right: 7px;
}

.line-height-1-2 {
  line-height: 1.2;
}

.lesson-parent {
  font-size: v-bind(fontSizeVW);
}

.exercise-title {
  text-align: left;
  margin-left: 6px;
  font-weight: bold;
}

.trainer-name {
  width: 100%;
  text-align: left;
  margin-left: 6px;
}

.start-end-time {
  width: 100%;
  text-align: left;
  margin-left: 6px;
}

.room-title {
  width: 100%;
  text-align: left;
  margin-left: 6px;
}

.td-container {
  padding: 0;
  height: 100%;
  position: relative;
  /* Важно добавить позиционирование */
}

.inner-div {
  height: 100%;
}

.lesson-parent {
  position: relative;
  border-radius: 10px;
  /* Закругление линии */
  padding-left: 8px;
  padding-right: 3px;
  padding-top: 3px;
  padding-bottom: 3px;
  background-color: #f0f0f0;
}

.lesson-color {
  position: absolute;
  left: 0;
  top: 0;
  width: 10px;
  height: 100%;
  border-top-left-radius: 10px;
  border-bottom-left-radius: 10px;
}
</style>
