<template>
  <div>
    <div class="page-title">
      <h3>Новая запись</h3>
    </div>

    <form class="form" @submit.prevent="submit">
      <div class="input-field" >
        <select v-model="category">
          <option
          v-for="c in categories"
          :key="c.id"
          :value="c.id"
          >{{c.name}}</option>
        </select>
        <label>Выберите категорию</label>
      </div>

      <p>
        <label>
          <input
          checked
              class="with-gap"
              name="type"
              type="radio"
              value="income"
              v-model="type"
          />
          <span>Доход</span>
        </label>
      </p>

      <p>
        <label>
          <input
              class="with-gap"
              name="type"
              type="radio"
              value="outcome"
              v-model="type"
          />
          <span>Расход</span>
        </label>
      </p>

      <div class="input-field">
        <input
            id="amount"
            type="number"
            v-model.number="amount"
            :class="{invalid: ($v.amount.$dirty && !$v.amount.required)}"
        >
        <label for="amount">Сумма</label>
        <span
          class="helper-text "
          :class="{invalid: ($v.amount.$dirty && !$v.amount.minValue) ||
              ($v.amount.$dirty && !$v.amount.required)}"
        >Amount</span>
      </div>

      <div class="input-field">
        <input
            id="description"
            type="text"
            v-model="description"
            :class="{invalid: ($v.description.$dirty && !$v.description.required)}"
        >
        <label for="description">Описание</label>
        <span
            class="helper-text"
            :class="{invalid: ($v.description.$dirty && !$v.description.required)}"
        >Description</span>
      </div>

      <button class="btn waves-effect waves-light" type="submit">
        Создать
        <i class="material-icons right">send</i>
      </button>
    </form>
  </div>
</template>

<script>
import { required, minValue } from 'vuelidate/lib/validators';
import { mapGetters } from 'vuex';

export default {
  data() {
    return {
      categories: [],
      category: null,
      type: 'income',
      amount: 1,
      description: null,
    };
  },

  validations: {
    type: { required },
    amount: { required, minValue: minValue(1) },
    description: {
      required,
    },
  },

  computed: {
    ...mapGetters(['info']),
    checkNegativeAmount() {
      if (this.type === 'income') {
        return true;
      }
      return this.info.bill >= this.amount;
    },
  },

  methods: {

    async submit() {
      if (this.$v.$invalid) {
        this.$v.$touch();
      } else if (this.checkNegativeAmount) {
        try {
          const dataRecord = {
            categoryId: this.category,
            type: this.type,
            amount: this.amount,
            description: this.description,
            date: new Date().toJSON(),
            // date: new Date(),
          };
          await this.$store.dispatch('addRecord', dataRecord);

          const bill = this.type === 'income'
            ? this.info.bill + this.amount
            : this.info.bill - this.amount;

          await this.$store.dispatch('updateInfo', { bill });
          this.$message('Запись созданна');
          this.amount = 1;
          this.description = '';
          this.$v.$reset();
          return;
        } catch (e) {
          console.log(e);
        }
      } else {
        this.$message(` Вам не хватило: ${this.amount - this.info.bill}`);
      }
    },
  },

  async mounted() {
    this.categories = await this.$store.dispatch('getListsCategory');
    // eslint-disable-next-line no-undef
    M.updateTextFields();

    if (this.categories.length) {
      this.category = this.categories[0].id;
    }
    // eslint-disable-next-line no-undef
    setTimeout(() => { M.AutoInit(); }, 0);
  },

};
</script>
