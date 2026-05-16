<template>
  <main class="container">
    <div class="left-section">
      <div class="cards-container">
        <div class="card card-front">
          <img src="@/assets/images/card-logo.svg" alt="Card Logo" class="card-logo" />
          <div class="card-number-display">{{ formattedDisplayCardNumber }}</div>
          <div class="card-details-display">
            <span class="card-name-display">{{ cardName || 'Jane Appleseed' }}</span>
            <span class="card-exp-display">{{ expMonth || '00' }}/{{ expYear || '00' }}</span>
          </div>
        </div>
        <div class="card card-back">
          <div class="cvc-display">{{ cvc || '000' }}</div>
        </div>
      </div>
    </div>

    <div class="right-section">
      <div v-if="!isCompleted" class="form-container">
        <form @submit.prevent="validateAndSubmit">
          <div class="input-group" :class="{ error: errors.cardName }">
            <label for="cardName">CARDHOLDER NAME</label>
            <input
              type="text"
              id="cardName"
              v-model="cardName"
              placeholder="e.g. Jane Appleseed"
              @input="clearError('cardName')"
            />
            <span class="error-msg" v-if="errors.cardName">{{ errors.cardName }}</span>
          </div>

          <div class="input-group" :class="{ error: errors.cardNumber }">
            <label for="cardNumber">CARD NUMBER</label>
            <input
              type="text"
              id="cardNumber"
              :value="cardNumber"
              @input="handleCardNumberInput"
              placeholder="e.g. 1234 5678 9123 0000"
              maxlength="19"
            />
            <span class="error-msg" v-if="errors.cardNumber">{{ errors.cardNumber }}</span>
          </div>

          <div class="form-row">
            <div class="input-group" :class="{ error: errors.expDate }">
              <label>EXP. DATE (MM/YY)</label>
              <div class="exp-inputs">
                <input
                  type="text"
                  v-model="expMonth"
                  placeholder="MM"
                  maxlength="2"
                  @input="handleExpMonthInput"
                />
                <input
                  type="text"
                  v-model="expYear"
                  placeholder="YY"
                  maxlength="2"
                  @input="handleExpYearInput"
                />
              </div>
              <span class="error-msg" v-if="errors.expDate">{{ errors.expDate }}</span>
            </div>

            <div class="input-group" :class="{ error: errors.cvc }">
              <label for="cvc">CVC</label>
              <input
                type="text"
                id="cvc"
                v-model="cvc"
                placeholder="e.g. 123"
                maxlength="3"
                @input="clearError('cvc')"
              />
              <span class="error-msg" v-if="errors.cvc">{{ errors.cvc }}</span>
            </div>
          </div>

          <button type="submit" class="btn-primary">Confirm</button>
        </form>
      </div>

      <div v-else class="success-container">
        <img src="@/assets/images/icon-complete.svg" alt="Complete Icon" class="complete-icon" />
        <h2>THANK YOU!</h2>
        <p>We've added your card details</p>
        <button class="btn-primary" @click="resetForm">Continue</button>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, computed } from 'vue';

const cardName = ref('');
const cardNumber = ref('');
const expMonth = ref('');
const expYear = ref('');
const cvc = ref('');

const isCompleted = ref(false);

const errors = ref({
  cardName: '',
  cardNumber: '',
  expDate: '',
  cvc: ''
});

const formattedDisplayCardNumber = computed(() => {
  const number = cardNumber.value.replace(/\s/g, '');
  if (!number) return '0000 0000 0000 0000';
  const padded = number.padEnd(16, '0');
  return padded.match(/.{1,4}/g).join(' ');
});

const handleCardNumberInput = (e) => {
  let val = e.target.value.replace(/\s/g, '');
  if (/[^\d]/.test(val)) {
    errors.value.cardNumber = 'Wrong format, numbers only';
  } else {
    errors.value.cardNumber = '';
  }

  if (val.length > 16) val = val.substring(0, 16);

  const matches = val.match(/.{1,4}/g);
  cardNumber.value = matches ? matches.join(' ') : val;
  e.target.value = cardNumber.value;
};

const handleExpMonthInput = (e) => {
  expMonth.value = e.target.value.replace(/[^\d]/g, '');
  clearError('expDate');
};

const handleExpYearInput = (e) => {
  expYear.value = e.target.value.replace(/[^\d]/g, '');
  clearError('expDate');
};

const clearError = (field) => {
  errors.value[field] = '';
};

const validateAndSubmit = () => {
  let isValid = true;

  if (!cardName.value.trim()) {
    errors.value.cardName = 'Can\'t be blank';
    isValid = false;
  }

  if (!cardNumber.value.trim()) {
    errors.value.cardNumber = 'Can\'t be blank';
    isValid = false;
  } else if (/[^\d\s]/.test(cardNumber.value)) {
    errors.value.cardNumber = 'Wrong format, numbers only';
    isValid = false;
  } else if (cardNumber.value.replace(/\s/g, '').length !== 16) {
    errors.value.cardNumber = 'Incomplete card number';
    isValid = false;
  }

  const currentYear = new Date().getFullYear() % 100;
  const currentMonth = new Date().getMonth() + 1;
  const monthNum = parseInt(expMonth.value, 10);
  const yearNum = parseInt(expYear.value, 10);

  if (!expMonth.value.trim() || !expYear.value.trim()) {
    errors.value.expDate = 'Can\'t be blank';
    isValid = false;
  } else if (!/^\d{2}$/.test(expMonth.value.padStart(2, '0')) || !/^\d{2}$/.test(expYear.value.padStart(2, '0'))) {
    errors.value.expDate = 'Wrong format';
    isValid = false;
  } else if (monthNum < 1 || monthNum > 12) {
    errors.value.expDate = 'Invalid month';
    isValid = false;
  } else if (yearNum < currentYear) {
    errors.value.expDate = 'Card expired';
    isValid = false;
  } else if (yearNum === currentYear && monthNum < currentMonth) {
    errors.value.expDate = 'Card expired';
    isValid = false;
  }

  if (!cvc.value.trim()) {
    errors.value.cvc = 'Can\'t be blank';
    isValid = false;
  } else if (!/^\d{3}$/.test(cvc.value)) {
    errors.value.cvc = 'Wrong format, 3 numbers';
    isValid = false;
  }

  if (isValid) {
    isCompleted.value = true;
  }
};

const resetForm = () => {
  cardName.value = '';
  cardNumber.value = '';
  expMonth.value = '';
  expYear.value = '';
  cvc.value = '';
  isCompleted.value = false;
  Object.keys(errors.value).forEach(k => errors.value[k] = '');
};
</script>

<style>
:root {
  --primary-red: hsl(0, 100%, 66%);
  --neutral-white: hsl(0, 100%, 100%);
  --neutral-light-gray: hsl(270, 3%, 87%);
  --neutral-dark-gray: hsl(212, 12%, 71%);
  --neutral-very-dark: hsl(278, 68%, 11%);
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Space Grotesk', sans-serif;
}

body {
  font-size: 18px;
  background-color: var(--neutral-white);
  color: var(--neutral-very-dark);
}

.container {
  display: flex;
  min-height: 100vh;
}

.left-section {
  width: 35%;
  max-width: 483px;
  background-image: url('~@/assets/images/bg-main-desktop.png');
  background-size: cover;
  background-position: center;
  position: relative;
  display: flex;
  align-items: center;
}

.cards-container {
  position: absolute;
  right: -250px;
  display: flex;
  flex-direction: column;
  gap: 30px;
  width: 540px;
}

.card {
  width: 447px;
  height: 245px;
  border-radius: 10px;
  position: relative;
  color: var(--neutral-white);
  box-shadow: 0 20px 40px rgba(0,0,0,0.1);
}

.card-front {
  background-image: url('~@/assets/images/bg-card-front.png');
  background-size: cover;
  padding: 30px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-self: flex-start;
  z-index: 2;
}

.card-back {
  background-image: url('~@/assets/images/bg-card-back.png');
  background-size: cover;
  align-self: flex-end;
}

.card-logo {
  width: 84px;
}

.card-number-display {
  font-size: 28px;
  letter-spacing: 3px;
  margin-top: 50px;
}

.card-details-display {
  display: flex;
  justify-content: space-between;
  text-transform: uppercase;
  font-size: 14px;
  letter-spacing: 2px;
}

.cvc-display {
  position: absolute;
  right: 55px;
  top: 110px;
  font-size: 14px;
  letter-spacing: 2px;
}

.right-section {
  width: 65%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.form-container, .success-container {
  width: 100%;
  max-width: 400px;
}

.form-container form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
  position: relative;
}

.form-row {
  display: flex;
  gap: 20px;
}

.form-row .input-group:first-child {
  flex: 1;
}

.form-row .input-group:last-child {
  flex: 1;
}

.exp-inputs {
  display: flex;
  gap: 10px;
}

label {
  font-size: 12px;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--neutral-very-dark);
}

input {
  width: 100%;
  height: 45px;
  border: 1px solid var(--neutral-light-gray);
  border-radius: 8px;
  padding: 0 15px;
  font-size: 18px;
  outline: none;
  cursor: pointer;
  color: var(--neutral-very-dark);
}

input::placeholder {
  color: var(--neutral-light-gray);
}

input:focus {
  border-color: transparent;
  background: linear-gradient(var(--neutral-white), var(--neutral-white)) padding-box,
              linear-gradient(to right, hsl(249, 99%, 64%), hsl(278, 94%, 30%)) border-box;
  border: 1px solid transparent;
}

.input-group.error input {
  border-color: var(--primary-red);
}

.error-msg {
  color: var(--primary-red);
  font-size: 12px;
  margin-top: 5px;
}

.btn-primary {
  background-color: var(--neutral-very-dark);
  color: var(--neutral-white);
  border: none;
  border-radius: 8px;
  height: 55px;
  font-size: 18px;
  cursor: pointer;
  margin-top: 10px;
  transition: opacity 0.3s;
}

.btn-primary:hover {
  opacity: 0.9;
}

.success-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  gap: 20px;
}

.complete-icon {
  width: 80px;
  margin-bottom: 10px;
}

.success-container h2 {
  letter-spacing: 3px;
  font-size: 28px;
}

.success-container p {
  color: var(--neutral-dark-gray);
  margin-bottom: 20px;
}

.success-container .btn-primary {
  width: 100%;
}

@media (max-width: 1024px) {
  .container {
    flex-direction: column;
  }

  .left-section {
    width: 100%;
    height: 250px;
    max-width: none;
    background-image: url('~@/assets/images/bg-main-mobile.png');
    align-items: flex-start;
  }

  .cards-container {
    position: absolute;
    top: 30px;
    right: auto;
    left: 50%;
    transform: translateX(-50%);
    flex-direction: column-reverse;
    align-items: stretch;
    width: 340px;
    margin-top: 0;
    gap: 0;
  }

  .card {
    width: 286px;
    height: 157px;
  }

  .card-front {
    margin-top: -70px;
    padding: 20px;
    z-index: 2;
    align-self: flex-start;
  }

  .card-back {
    align-self: flex-end;
  }

  .card-logo {
    width: 54px;
  }

  .card-number-display {
    font-size: 18px;
    letter-spacing: 2px;
    margin-top: 30px;
  }

  .card-details-display {
    font-size: 10px;
  }

  .cvc-display {
    right: 35px;
    top: 72px;
    font-size: 10px;
  }

  .right-section {
    width: 100%;
    padding: 80px 20px;
  }
}
</style>
