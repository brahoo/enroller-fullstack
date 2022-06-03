<template>
  <div id="app">
    <h1>
      <img src="./assets/logo.svg" alt="Enroller" class="logo">
      System do zapisów na zajęcia
    </h1>
    <div v-if="authenticatedUsername">
      <h2>Witaj {{ authenticatedUsername }}!
        <a @click="logout()" class="float-right  button-outline button">Wyloguj</a>
      </h2>
      <meetings-page :username="authenticatedUsername"></meetings-page>
    </div>
    <div v-else>
	  <div :class = " ' alert alert-' + (this.isError ? 'error' : 'success')" v-if  = "message">{{message}}</div>
      <button @click = "registering = false" :class = " registering ? 'button-outline' : ''">Zaloguj</button>
      <button @click = "registering = true" :class = " !registering ? 'button-outline' : ''">Zarejestruj</button>	
      <login-form @submit = "registering ?  regiser($event) : login($event)"  :button-label = "loginButtonLabel" ></login-form>
    </div>
  </div>
</template>

<script>
    import "milligram";
    import LoginForm from "./LoginForm";
    import MeetingsPage from "./meetings/MeetingsPage";

    export default {
        components: {LoginForm, MeetingsPage},
        data() {
            return {
                authenticatedUsername: "",
                registering: false,
				message: '',
				isError: false
            };
        },
        methods: {
            login(user) {
				this.message = undefined;
				this.$http.post('tokens', user)
					.then(response =>{
						const token = response.body.token;
						this.storeAuth(user.login, token);
					})
					.catch(response => {
						this.message = "Logowanie nie powiodło się! Kod odpowiedzi: " + response.status;
						this.isError = true;
					});	
            },
			storeAuth(username, token) {
				this.authenticatedUsername = username;
				Vue.http.headers.common.Authorization = 'Bearer ' + token;
				localStorage.setItem('username', username);
				localStorage.setItem('token', token);
			},
            logout() {
                this.authenticatedUsername = '';
            },
			regiser(user) {
				this.message = undefined;
				this.$http.post('participants', user)
					.then(response => {
						this.message = 'Rejestracja powiodła się.';
						this.isError = false;
					})
					.catch(response => {
						this.message = "Rejestracja nie powiodła się! Kod odpowiedzi: " + response.status;
						this.isError = true;
					});
			},
        },
		computed: {
			loginButtonLabel() {
				return this.registering ? 'Zarejestruj się' : 'Zaloguj się';
			}
		}
    };
</script>

<style>
  #app {
    max-width: 1000px;
    margin: 0 auto;
  }

  .logo {
    vertical-align: middle;
  }
  
  .alert {
    padding: 10px;
    margin-bottom: 10px;
    border: 2px solid black;
  }
  
  .alert-success {
	background: lightgreen;
	border-color: darken(lightgreen,10%);
  }
  
  .alert-error {
	background: indianred;
	border-color: darken(indianred,10%);
  }
</style>

