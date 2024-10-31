<template>
  <v-app id="login" class="secondary">
    <v-content>
      <v-container fluid fill-height>
        <v-layout align-center justify-center>
          <v-flex xs12 sm8 md4 lg4>
            <v-card class="elevation-1 pa-3">
              <v-card-text>
                <div class="layout column align-center">
                  <img src="static/logo.png" alt="Vue Material Admin" width="180" height="180">
                  <h1 class="flex my-4 primary--text">Referral System</h1>
                </div>
                <v-form>
                  <v-text-field
                    append-icon="person"
                    name="login"
                    label="Login"
                    type="text"
                    v-model="userEmail"
                    :error="error"
                    :rules="[rules.required]"/>

                  <v-text-field
                    :type="hidePassword ? 'password' : 'text'"
                    :append-icon="hidePassword ? 'visibility_off' : 'visibility'"
                    name="password"
                    label="Password"
                    id="password"
                    :rules="[rules.required]"
                    v-model="password"
                    :error="error"
                    @click:append="hidePassword = !hidePassword"/>
                </v-form>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn block color="primary" @click="login" :loading="loading" :disabled="loading">
                  <template v-if="loading">
                    <span> Processing...</span>
                  </template>
                  <template v-else>
                    Login
                  </template>
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
      <v-snackbar
        v-model="showResult"
        :timeout="2500"
        top>
        {{ result }}
      </v-snackbar>
    </v-content>
  </v-app>
</template>

<script>
  import { setUrl } from '../../config/setUrl'; // Importa la URL de la API
  const API_URL = setUrl(); // Aquí cargo el valor de la URL

  export default {
    data() {
      return {
        loading: false,
        userEmail: '',
        password: '',
        hidePassword: true,
        error: false,
        showResult: false,
        result: '',
        rules: {
          required: value => !!value || 'Required.'
        }
      }
    },

    methods: {
      async login() {
        console.log('Function login ***');

        // Retorna si no hay email o password
        if (!this.userEmail || !this.password) {
          this.result = "Login and Password can't be null.";
          this.showResult = true;
          return;
        }

        // Deshabilitar el botón de inicio de sesión
        this.loading = true;

        // Configuración de la petición
        const payload = {
          username: this.userEmail,
          password: this.password
        };

        try {
          // Simular un tiempo de procesamiento
          await new Promise(resolve => setTimeout(resolve, 500)); // Espera 500 ms antes de hacer la solicitud

          // Hacer la petición POST a la API
          const response = await fetch(`${API_URL}/users/login`, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
            },
            body: JSON.stringify(payload),
          });

          // Verificar si la respuesta es exitosa
          if (!response.ok) {
            const errorData = await response.json();
            console.log(JSON.stringify(errorData))
            this.error = true;
            this.result = errorData.message || errorData.errors;
            this.showResult = true;
            return;
          }

          // Obtener el token de la respuesta
          const data = await response.json();
          // Aquí puedes almacenar el token en el almacenamiento local o de sesión
          localStorage.setItem('token', data.token);

          // Redirigir a la página del Dashboard
          this.$router.push({ name: 'Dashboard' });
        } catch (error) {
          console.error('Error en la petición:', error);
          this.error = true;
          this.result = 'Error during login';
          this.showResult = true;
        } finally {
          // Volver a habilitar el botón de inicio de sesión
          this.loading = false;
        }
      }
    }
  }
</script>

<style scoped lang="css">
  #login {
    height: 50%;
    width: 100%;
    position: absolute;
    top: 0;
    left: 0;
    content: "";
    z-index: 0;
  }
</style>
