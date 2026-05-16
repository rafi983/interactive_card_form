# Frontend Mentor - Interactive card details form solution

This is a solution to the [Interactive card details form challenge on Frontend Mentor](https://www.frontendmentor.io/challenges/interactive-card-details-form-XpS8cKZDWw). 

## Table of contents

- [Overview](#overview)
  - [screenshot](#screenshot)
  - [Links](#links)
- [My process](#my-process)
  - [Built with](#built-with)
  - [What I learned](#what-i-learned)
  - [Continued development](#continued-development)
- [Author](#author)

## Overview

### The challenge

Users should be able to:

- Fill in the form and see the card details update in real-time
- Receive error messages when the form is submitted if:
  - Any input field is empty
  - The card number, expiry date, or CVC fields are in the wrong format
  - The card expiration date is in the past (custom extended validation)
- View the optimal layout depending on their device's screen size
- See hover, active, and focus states for interactive elements on the page

### Screenshot

![](.public/design/desktop-design.jpg)

### Links

- Solution URL: [GitHub Repository](https://github.com/yourusername/vue-interactive-card-form) <!-- Update this with your actual repository URL -->
- Live Site URL: [Live Site](https://yourusername.github.io/vue-interactive-card-form) <!-- Update this with your actual live URL -->

## My process

### Built with

- Semantic HTML5 markup
- CSS custom properties
- CSS Flexbox
- Mobile-first workflow
- [Vue.js 3](https://vuejs.org/) - JS framework (Composition API / `<script setup>`)

### What I learned

This project provided an excellent opportunity to practice reactive state management with Vue 3's Composition API and handle complex form validation manually.

I learned how to efficiently format the credit card number visually using computed properties, generating the spaces automatically:

```js
const formattedDisplayCardNumber = computed(() => {
  const number = cardNumber.value.replace(/\s/g, '');
  if (!number) return '0000 0000 0000 0000';
  const padded = number.padEnd(16, '0');
  return padded.match(/.{1,4}/g).join(' ');
});
```\n\nI also improved my skills with detailed custom form validations, for instance checking the expiry month and year cooperatively against the user's local system time to make sure the card is valid.\n\n### Continued development\n\nGoing forward, I would like to focus on further abstracting form inputs into reusable Vue UI components and integrating robust validation libraries like VeeValidate or Yup.\n\n## Author\n\n- Frontend Mentor - [@rafi983](https://www.frontendmentor.io/profile/rafi983)\n