<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { useAuth } from '@/composables/useAuth.js';
import { auth } from '@/firebase.js';
import BaseButton from '@/components/ui/BaseButton.vue';

const { 
  authError, 
  signInWithGoogle, 
  sendSignInLink,
  isSignInWithEmailLink,
  signInWithEmailLink
} = useAuth();

const router = useRouter();

const email = ref('');
const isSubmitting = ref(false);
const linkSent = ref(false);

onMounted(async () => {
  if (isSignInWithEmailLink(auth, window.location.href)) {
    let storedEmail = window.localStorage.getItem('emailForSignIn');
    if (!storedEmail) {
      storedEmail = window.prompt('Пожалуйста, введите ваш email для завершения входа.');
    }
    
    if (storedEmail) {
      isSubmitting.value = true;
      try {
        await signInWithEmailLink(auth, storedEmail, window.location.href);
        window.localStorage.removeItem('emailForSignIn');
        router.push('/');
      } catch (error) {
        console.error("Ошибка при входе по ссылке:", error);
        authError.value = "Неверная или истекшая ссылка. Попробуйте снова.";
      } finally {
        isSubmitting.value = false;
      }
    }
  }
});

const handleSendLink = async () => {
    isSubmitting.value = true;
    authError.value = null;
    linkSent.value = false;
    
    const success = await sendSignInLink(email.value);
    
    if (success) {
        linkSent.value = true;
    }
    
    isSubmitting.value = false;
}

const handleGoogleSignIn = async () => {
    await signInWithGoogle();
    router.push('/');
}
</script>

<template>
  <main class="py-10 md:py-25">
    <div class="max-w-md mx-auto">
      <div class="bg-white p-8 rounded-3xl shadow-lg">
        
        <h2 class="text-h3-panda font-bold text-center mb-2">Вход или регистрация</h2>
        <p class="text-center text-dark-gray mb-8">
            Введите ваш email, чтобы войти или создать аккаунт
        </p>

        <div v-if="linkSent" class="text-center p-4 bg-green-100 text-green-800 rounded-lg mb-6">
            <h3 class="font-bold">Ссылка отправлена!</h3>
            <p>Проверьте вашу почту <span class="font-semibold">{{ email }}</span> и перейдите по ссылке для завершения входа.</p>
        </div>
        
        <form v-else @submit.prevent="handleSendLink" class="space-y-6">
            <div class="form-group">
                <div class="form-control">
                <input type="email" placeholder="Email" required v-model.trim="email">
                <span class="input-border"></span>
                </div>
            </div>
            <div v-if="authError" class="error-message text-center">
                {{ authError }}
            </div>
            <BaseButton type="submit" :disabled="isSubmitting" class="w-full">
                <span v-if="!isSubmitting">Получить ссылку для входа</span>
                <span v-else>Отправка...</span>
            </BaseButton>
        </form>

        <div class="flex items-center my-6">
          <hr class="flex-grow border-t border-gray">
          <span class="mx-4 text-sm text-dark-gray">или</span>
          <hr class="flex-grow border-t border-gray">
        </div>

        <button @click="handleGoogleSignIn" class="w-full flex items-center justify-center gap-3 py-3 px-4 border border-gray rounded-full hover:bg-light-gray transition-colors">
            <img src="@/assets/images/pages/AuthPage/google-gradient-icon.svg" alt="Google" class="w-7 h-7">
            <span class="text-panda-black font-semibold text-sm">Войти через Google</span>
        </button>

      </div>
    </div>
  </main>
</template>

<style scoped>
/* Стили остаются такими же, как и были */
.form-group .error-message {
  color: #F15F31;
  font-size: 14px;
  margin-top: 4px;
  min-height: 20px;
}
input {
  font-family: 'Gilroy-Medium', sans-serif;
  font-size: 16px;
  width: 100%;
  border: none;
  border-bottom: 1px solid #E3E3E3;
  padding: 10px 4px;
  color: #131C26;
  background-color: transparent;
  transition: background-color 0.2s ease;
  position: relative;
  z-index: 1;
}
input::placeholder { color: #8F8F8F; }
input:focus { outline: none; }
input:hover {
  background-color: rgba(227, 227, 227, 0.2);
}
.form-control {
  position: relative;
}
.input-border {
  position: absolute;
  background: #F15F31;
  width: 0%;
  height: 2px;
  bottom: 0;
  left: 0;
  transition: width 0.3s ease-in-out;
  z-index: 2;
}
input:focus ~ .input-border {
  width: 100%;
}
</style>