<template>
  <v-row class="d-flex justify-center align-center login--container">
    <v-col cols="3">
      <v-card class="pa-3">
        <v-card-title class="d-flex justify-center mt-5 mb-5">
          <v-img src="/talently-logo.png" max-width="150"></v-img>
        </v-card-title>
        <v-card-text>
          <v-form ref="form" v-model="valid" lazy-validation>
            <div class="mb-2">
              <span class="black--text">Email</span>
            </div>
            <v-text-field
              dense
              v-model="user.email"
              outlined
              :rules="emailRules"
              type="email"
            ></v-text-field>
            <div class="d-flex justify-space-between mb-2">
              <span class="black--text">Password</span>
              <span class="password-recovery font-weight-bold"
                >¿Se te olvidó tu contraseña?</span
              >
            </div>
            <v-text-field
              @keydown="keyDownListener"
              dense
              v-model="user.password"
              outlined
              :rules="isRequired"
              type="password"
            ></v-text-field>

            <v-alert type="error" v-if="error">
              Usuario y/o contraseña invalidos.
            </v-alert>
            <v-btn
              @click="userLogin"
              class="text-none btn--text"
              :loading="loading"
              large
              depressed
              block
              color="#30308C"
              dark
              >Iniciar sesión</v-btn
            >
            <p class="text-center mt-5 create-account">
              <span>¿Nuevo en Talently?</span>

              <NuxtLink to="/create-account" class="create-account--link"
                >Crea una cuenta.</NuxtLink
              >
            </p>
          </v-form>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  layout: "session",
  data: () => ({
    user: {
      email: "",
      password: "",
    },
    valid: true,
    emailRules: [
      (v) => !!v || "Campo requerido",
      (v) => /.+@.+\..+/.test(v) || "Ingresa un formato de email válido",
    ],
    isRequired: [(v) => !!v || "Campo requerido"],
    loading: false,
    error: false,
  }),
  methods: {
    async userLogin() {
      if (!this.$refs.form.validate()) return;

      try {
        this.loading = true;
        const { status } = await this.$auth.loginWith("local", {
          data: this.user,
        });

        const { data } = await this.$axios.post("content");
        this.$store.dispatch("listContentAdd", data.content);

        if (status === 200) this.$router.push("/");
      } catch (err) {
        this.error = true;

        setTimeout(() => {
          this.error = false;
        }, 3000);
      } finally {
        this.loading = false;
      }
    },
    keyDownListener(e) {
      if (e.key === "Enter") this.userLogin();
    },
  },
};
</script>

<style lang="scss" scoped>
.login--container {
  height: 100vh;
  background-color: #ecf1f6;
}

.password-recovery {
  color: #30308c;
  font-size: 0.95rem;
}

.btn--text {
  font-size: 14px;
  font-weight: 600;
  letter-spacing: normal;
}

.create-account {
  font-size: 12px;
  color: #000000;
}

.create-account--link {
  color: #30308c;
  font-weight: 700;
}
</style>