<template>
  <fieldset class="agenda-item-form">
    <button type="button" class="agenda-item-form__remove-button" @click="cleanAgenda">
      <ui-icon icon="trash" />
    </button>

    <ui-form-group>
      <ui-dropdown title="Тип" :options="$options.agendaItemTypeOptions" name="type" v-model="localAgendaItem.type" />
    </ui-form-group>

    <div class="agenda-item-form__row">
      <div class="agenda-item-form__col">
        <ui-form-group label="Начало">
          <ui-input
            type="time"
            placeholder="00:00"
            name="startsAt"
            v-model="localAgendaItem.startsAt"
          />
        </ui-form-group>
      </div>
      <div class="agenda-item-form__col">
        <ui-form-group label="Окончание">
          <ui-input
            type="time"
            placeholder="00:00"
            name="endsAt"
            v-model="localAgendaItem.endsAt"
          />
        </ui-form-group>
      </div>
    </div>

    <ui-form-group :label="setTitle">
      <ui-input name="title"
                v-model="localAgendaItem.title"
                @change="
              $emit('update:agendaItem', {
                ...agendaItem,
                title: $event.target.value,
              })
            "
      />
    </ui-form-group>
    <ui-form-group v-if="localAgendaItem.type == 'talk'" label="Докладчик">
      <ui-input
        name="speaker"
        v-model="localAgendaItem.speaker"
      />
    </ui-form-group>
    <ui-form-group v-if="localAgendaItem.type == 'talk' || localAgendaItem.type == 'other'"  label="Описание">
      <ui-input multiline
                name="description"
                v-model="localAgendaItem.description"
      />
    </ui-form-group>
    <ui-form-group v-if="localAgendaItem.type == 'talk'" label="Язык">
      <ui-dropdown
        title="Язык"
        :options="$options.talkLanguageOptions"
        name="language"
        v-model="localAgendaItem.language"

      />
    </ui-form-group>
  </fieldset>
</template>

<script>
import UiIcon from './UiIcon';
import UiFormGroup from './UiFormGroup';
import UiInput from './UiInput';
import UiDropdown from './UiDropdown';



const agendaItemTypeIcons = {
  registration: 'key',
  opening: 'cal-sm',
  talk: 'tv',
  break: 'clock',
  coffee: 'coffee',
  closing: 'key',
  afterparty: 'cal-sm',
  other: 'cal-sm',
};

const agendaItemDefaultTitles = {
  registration: 'Регистрация',
  opening: 'Открытие',
  break: 'Перерыв',
  coffee: 'Coffee Break',
  closing: 'Закрытие',
  afterparty: 'Afterparty',
  talk: 'Доклад',
  other: 'Другое',
};

const agendaItemTypeOptions = Object.entries(agendaItemDefaultTitles).map(([type, title]) => ({
  value: type,
  text: title,
  icon: agendaItemTypeIcons[type],
}));

const talkLanguageOptions = [
  { value: null, text: 'Не указано' },
  { value: 'RU', text: 'RU' },
  { value: 'EN', text: 'EN' },
];

export default {
  name: 'MeetupAgendaItemForm',

  agendaItemTypeOptions,
  talkLanguageOptions,

  components: { UiIcon, UiFormGroup, UiInput, UiDropdown },

  props: {
    agendaItem: {
      type: Object,
      required: true,
    },
  },
  data(){
    return {
      localAgendaItem: { ...this.agendaItem },


    }
  },

  emits:['update:agendaItem', 'remove'],
  computed:{
        setTitle() {
      if(this.localAgendaItem.type == 'talk'){
        return 'Тема'
      } else if (this.localAgendaItem.type == 'other'){
        return 'Заголовок'
      } else {
        return 'Нестандартный текст (необязательно)'
      }
    }

  },
  methods: {
    cleanAgenda(){
      this.$emit('remove');
    }
  },
 watch: {
   localAgendaItem: {
     deep: true,
     handler() {
       this.$emit('update:agendaItem', {...this.localAgendaItem});
     }
   },
    'localAgendaItem.startsAt':{

      deep: true,
      handler(newVal, oldVal) {
      let end_hms = this.localAgendaItem.endsAt;
      let start_point_old = new Date("1970-01-01T" + newVal);
      let start_point_new = new Date("1970-01-01T" + oldVal);
      let end_point = new Date("1970-01-01T" + end_hms);

      let difference = Math.abs(start_point_new.getTime() - start_point_old.getTime()) / 3600000;
        let new_end_point_hours = end_point.getHours()
             if (newVal > oldVal) {
                new_end_point_hours = end_point.getHours() + difference
        if (new_end_point_hours > 24){
          new_end_point_hours -= 24;
        }

      } else if (newVal < oldVal) {
                new_end_point_hours = end_point.getHours() - difference
        if (new_end_point_hours > 24){
          new_end_point_hours -= 24;
        }
      }
      this.localAgendaItem.endsAt = String(new_end_point_hours).padStart(2, "0")  +':'+ String(end_point.getMinutes()).padStart(2, "0")
      }
    }
 }
};
</script>

<style scoped>
.agenda-item-form {
  border: 2px solid var(--blue-light);
  border-radius: 8px;
  position: relative;
  padding: 20px 10% 0 16px;
}

.agenda-item-form__remove-button {
  position: absolute;
  top: 4px;
  right: 0;
  box-shadow: none;
  border: none;
  background-color: transparent;
  outline: none;
  padding: 4px;
  cursor: pointer;
  transition: 0.2s opacity;
}

.agenda-item-form__remove-button:hover {
  opacity: 0.6;
}

.agenda-item-form__row {
  display: flex;
  flex-direction: column;
}

.agenda-item-form__col + .agenda-item-form__col {
  margin-top: 16px;
}

.agenda-item-form__col:first-child {
  margin-left: 0;
}

@media all and (min-width: 992px) {
  .agenda-item-form {
    padding: 28px 25% 4px 24px;
  }

  .agenda-item-form__remove-button {
    top: 20px;
    right: 20px;
  }

  .agenda-item-form__row {
    flex-direction: row;
    justify-content: space-between;
    margin: 0 -12px;
  }

  .agenda-item-form__col {
    flex: 1 1 auto;
    padding: 0 12px;
  }

  .agenda-item-form__col + .agenda-item-form__col {
    margin-top: 0;
  }

  .agenda-item-form__col:first-child {
    margin-left: 0;
  }
}
</style>
