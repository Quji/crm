<template>
  <form class="card auth-card" @submit.prevent="submitHandler">
    <div class="card-content">
      <span class="card-title">Домашняя бухгалтерия</span>
      <div class="input-field">
        <input
            id="email"
            type="text"
            v-model.trim = "email"
            :class="{invalid: ($v.email.$dirty && !$v.email.required)
            || ($v.email.$dirty && !$v.email.email) }"
            >
        <label for="email">Email</label>
        <small
        class="helper-text invalid"
        v-if="$v.email.$dirty && !$v.email.required"
        >
        Поле не должно быть пустым
        </small>
        <small
        class="helper-text invalid"
        v-if="$v.email.$dirty && !$v.email.email"
        >
        Введите корректную почту
        </small>
      </div>
      <div class="input-field">
        <input
            id="password"
            type="password"
            class="validate"
            v-model="password"
            :class="{invalid: ($v.password.$dirty && !$v.password.required)
            || ($v.password.$dirty && !$v.password.minLength) }"
        >
        <label for="password">Пароль</label>
        <small class="helper-text invalid"
        v-if="$v.password.$dirty && !$v.password.required"
        >Не должен быть пустым</small>
        <small class="helper-text invalid"
        v-if="$v.password.$dirty && !$v.password.minLength"
        > Не менее {{this.$v.password.$params.minLength.min}}
        симолов, сейчас {{password.length}} </small>
      </div>
    </div>
    <div class="card-action">
      <div>
        <button
            class="btn waves-effect waves-light auth-submit"
            type="submit"
        >
          Войти
          <i class="material-icons right">send</i>
        </button>
      </div>

      <p class="center">
        Нет аккаунта?
        <router-link to="/register">Зарегистрироваться</router-link>
      </p>
    </div>
  </form>
</template>

<script>
import { email, required, minLength } from 'vuelidate/lib/validators';
import messages from '@/plugins/message';

export default {
  name: 'login',
  data() {
    return {
      email: '',
      password: '',
    };
  },
  validations: {
    email: { email, required },
    password: { required, minLength: minLength(6) },
  },
  mounted() {
    if (messages[this.$route.query.message]) this.$message(messages[this.$route.query.message]);
  },
  methods: {
    async submitHandler() {
      if (this.$v.$invalid) {
        this.$v.$touch();
        return;
      }
      const formData = {
        email: this.email,
        password: this.password,
      };

      // eslint-disable-next-line no-useless-catch
      try {
        await this.$store.dispatch('login', formData);
        this.$router.push('/');
      // eslint-disable-next-line no-empty
      } catch (e) {}
    },

  },

};
</script>
