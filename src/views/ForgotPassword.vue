<template>
  <div class="reset-password">
    <Modal v-if="modalActive" :modalMessage="modalMessage" v-on:close-modal="closeModal"/>
    <Loading v-if="loading"/>
    <div class="form-wrap">
        <form class="reset">
          <p class="login-register">
            Už máte účet?
            <router-link class="router-link" :to="{name: 'Prihlasit'}">Přihlásit se!</router-link>
          </p>
            <h2>Resetování hesla!</h2>
            <p>Zapomněli jste heslo? Zadejte email a resetujte si ho!</p>
            <div class="inputs">
                <div class="input">
                    <input type="text" placeholder="Email" v-model="email">
                    <email class="icon" />
                </div>
            </div>
            <button @click.prevent="resetPassword">Resetovat</button>
            <div class="angle"></div>
        </form>
        <div class="background"></div>
    </div>
  </div>
</template>

<script>
import email from "../assets/Icons/envelope-regular.svg";
import Modal from "../components/Modal.vue";
import Loading from "../components/Loading.vue";
import firebase from "firebase/app";
import "firebase/auth";

export default {
    name: "ForgotPassword",
    data(){
        return{
            email: "",
            modalActive: false,
            modalMessage: "",
            loading: null,
        }
    },
    components: {
    email,
    Modal,
    Loading,
},
    methods: {
       resetPassword() {
        this.loading = true;
        firebase
          .auth()
          .sendPasswordResetEmail(this.email)
          .then(() => {
            this.modalMessage = "Jestli váš účet existuje, dostanete email s odkazem pro resetování vašeho hesla!";
            this.loading = false;
            this.modalActive = true;
          })
          .catch((err) => {
            this.modalMessage = err.message;
            this.loading = false;
            this.modalActive = true;
          });
      },
        closeModal(){
            this.modalActive = !this.modalActive;
            this.email = "";
        }
    }
};
</script>

<style lang="scss" scoped>
.reset-password {
  position: relative;
  .form-wrap {
    .reset {
      h2 {
        margin-bottom: 8px;
      }
      p {
        text-align: center;
        margin-bottom: 32px;
      }
    }
  }
}
</style>