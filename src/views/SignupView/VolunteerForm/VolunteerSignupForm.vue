<template>
  <form
    v-if="step == 'step-1'"
    class="uc-form-body"
    @submit.prevent="nextPage()"
  >
    <div v-if="errors.length" class="step-errors">
      <h5>Please correct the following problems:</h5>
      <ul>
        <li v-for="error in errors" v-bind:key="error">{{ error }}</li>
      </ul>
    </div>

    <div class="step-title">Step 1 of 2: Choose your log-in details</div>

    <div class="uc-column">
      <label for="inputEmail" class="uc-form-label">What's your email?</label>
      <input
        id="inputEmail"
        type="email"
        class="uc-form-input"
        v-bind:class="{
          'uc-form-input--invalid': invalidInputs.indexOf('inputEmail') > -1
        }"
        v-model="credentials.email"
        required
        autofocus
        autocomplete="email"
      />
      <p class="uc-form-subtext">
        We will only use your email to contact you about your account. See our
        Privacy Policy for more info.
      </p>
    </div>

    <div class="uc-column">
      <label for="inputPassword" class="uc-form-label">
        Create a password.
      </label>
      <input
        id="inputPassword"
        type="password"
        class="uc-form-input"
        v-bind:class="{
          'uc-form-input--invalid': invalidInputs.indexOf('inputPassword') > -1
        }"
        v-model="credentials.password"
        required
        autocomplete="new-password"
      />
      <p class="uc-form-subtext">
        Keep your account safe by choosing a password with one number, one
        uppercase letter, and one lowercase letter.
      </p>
    </div>

    <button class="uc-form-button" type="submit">
      Continue
    </button>

    <div v-if="msg !== ''">{{ msg }}</div>
  </form>

  <form
    v-else-if="step == 'step-2'"
    class="uc-form-body"
    @submit.prevent="checkInputs($event)"
  >
    <div v-if="errors.length" class="step-errors">
      <h5>Please correct the following problems:</h5>
      <ul>
        <li v-for="error in errors" v-bind:key="error">{{ error }}</li>
      </ul>
    </div>

    <div class="step-title">Step 2 of 2: Tell us about yourself!</div>

    <div class="uc-column">
      <!-- Fix for bug in Chrome where the first two fields are parsed as a username and password
       even if the HTML5 autocomplete attributes are set to the right values -->
      <label for="username" class="d-none">Username</label>
      <input
        type="text"
        class="d-none"
        id="username"
        v-model="credentials.email"
        autocomplete="username"
      />
      <label for="password" class="d-none">Password</label>
      <input
        type="password"
        class="d-none"
        id="password"
        v-model="credentials.password"
        autocomplete="new-password"
      />

      <label for="firstName" class="uc-form-label">First Name</label>
      <input
        id="firstName"
        type="text"
        class="uc-form-input"
        v-bind:class="{
          'uc-form-input--invalid': invalidInputs.indexOf('firstName') > -1
        }"
        v-model="profile.firstName"
        required
        autofocus
        autocomplete="given-name"
      />
    </div>

    <div class="uc-column">
      <label for="lastName" class="uc-form-label">Last Name</label>
      <input
        id="lastName"
        type="text"
        class="uc-form-input"
        v-bind:class="{
          'uc-form-input--invalid': invalidInputs.indexOf('lastName') > -1
        }"
        v-model="profile.lastName"
        required
        autocomplete="family-name"
      />
    </div>

    <div class="uc-column">
      <label for="phoneNumber" class="uc-form-label">Phone Number</label>
      <vue-phone-number-input
        id="phoneNumber"
        class="phone-input"
        v-model="profile.phone"
        :required="true"
        :error="invalidInputs.indexOf('phone') > -1 && !phoneInputInfo.isValid"
        color="#555"
        valid-color="#16ba97"
        @update="onPhoneInputUpdate"
      />
    </div>

    <div class="uc-column">
      <label for="college" class="uc-form-label">
        College
      </label>
      <input
        id="college"
        type="text"
        class="uc-form-input"
        v-bind:class="{
          'uc-form-input--invalid': invalidInputs.indexOf('college') > -1
        }"
        v-model="profile.college"
        required
      />
    </div>

    <div class="uc-column">
      <label for="favoriteAcademicSubject" class="uc-form-label">
        What’s your favorite academic subject?
      </label>
      <input
        id="favoriteAcademicSubject"
        type="text"
        class="uc-form-input"
        v-bind:class="{
          'uc-form-input--invalid':
            invalidInputs.indexOf('favoriteAcademicSubject') > -1
        }"
        v-model="profile.favoriteAcademicSubject"
        required
      />
    </div>

    <div class="uc-form-checkbox">
      <input
        id="userAgreement"
        v-model="credentials.terms"
        type="checkbox"
        required
      />
      <label for="userAgreement">
        I have read and accept the
        <a href="/legal" target="_blank">user agreement</a>.
      </label>
    </div>

    <button class="uc-form-button" type="submit">
      Sign Up
    </button>

    <!-- <div v-if="msg !== ''">{{ msg }}</div> -->
  </form>

  <div v-else-if="step == 'success-message'" class="uc-form-body">
    You've been sent a verification email! Check your email for a link to
    confirm your account.
  </div>

  <div v-else class="uc-form-body">Unexpected Error</div>
</template>

<script>
import validator from "validator";
import * as Sentry from "@sentry/browser";

import AuthService from "@/services/AuthService";
import RegistrationService from "@/services/RegistrationService";
import UserService from "@/services/UserService";
import AnalyticsService from "@/services/AnalyticsService";

export default {
  data() {
    return {
      msg: "",
      errors: [],
      invalidInputs: [],
      credentials: {
        email: "",
        password: "",
        terms: false
      },
      profile: {
        firstName: "",
        lastName: "",
        college: "",
        phone: "",
        favoriteAcademicSubject: ""
      },
      step: "step-1",
      phoneInputInfo: {}
    };
  },
  methods: {
    nextPage() {
      // validate input
      this.errors = [];
      this.invalidInputs = [];
      if (!this.credentials.email) {
        this.errors.push("An email address is required.");
        this.invalidInputs.push("inputEmail");
      } else if (!validator.isEmail(this.credentials.email)) {
        // this is necessary because browsers ignore <input type="email"> until the
        // user actually tries to submit the form, which does not occur until step 2
        this.errors.push(
          this.credentials.email + " is not a valid email address."
        );
        this.invalidInputs.push("inputEmail");
      }
      if (!this.credentials.password) {
        this.errors.push("A password is required.");
        this.invalidInputs.push("inputPassword");
      }
      if (this.errors.length) {
        return;
      }

      // check credentials
      AuthService.checkRegister(this, {
        email: this.credentials.email,
        password: this.credentials.password
      })
        .then(() => {
          this.step = "step-2";
        })
        .catch(err => {
          this.msg = err.message;
          if (err.status !== 409 && err.status !== 422) {
            Sentry.captureException(err);
          }
        });
    },
    checkInputs() {
      this.errors = [];
      this.invalidInputs = [];

      // validate input
      if (!this.profile.firstName || !this.profile.lastName) {
        this.errors.push("You must enter your first and last name.");
      }
      if (!this.profile.firstName) {
        this.invalidInputs.push("firstName");
      }
      if (!this.profile.lastName) {
        this.invalidInputs.push("lastName");
      }
      if (!this.profile.phone) {
        this.errors.push("You must enter a phone number.");
        this.invalidInputs.push("phone");
      } else if (!this.phoneInputInfo.isValid || !this.phoneInputInfo.e164) {
        this.errors.push(this.profile.phone + " is not a valid phone number.");
        this.invalidInputs.push("phone");
      }
      if (!this.profile.college) {
        this.errors.push("Please enter the name of the college you go to.");
        this.invalidInputs.push("college");
      }
      if (!this.profile.favoriteAcademicSubject) {
        this.errors.push("Please enter your favorite academic subject.");
        this.invalidInputs.push("favoriteAcademicSubject");
      }
      if (!this.credentials.terms) {
        this.errors.push("You must read and accept the user agreement.");
      }
      if (!this.errors.length) {
        this.submit();
      }
    },
    submit() {
      AuthService.register(this, {
        isVolunteer: true,
        code: RegistrationService.data.registrationCode,
        email: this.credentials.email,
        password: this.credentials.password,
        terms: this.credentials.terms,
        firstName: this.profile.firstName,
        lastName: this.profile.lastName,
        phone: this.phoneInputInfo.e164,
        college: this.profile.college,
        favoriteAcademicSubject: this.profile.favoriteAcademicSubject
      })
        .then(() => UserService.getUser())
        .then(user => {
          UserService.setProfile(this, user).catch(err => {
            this.msg = err.message;
            Sentry.captureException(err);
          });

          // analytics: tracking when a user has signed up
          AnalyticsService.identify(user, user.isFakeUser);
          AnalyticsService.trackNoProperties("signed up", user.isFakeUser);

          this.step = "success-message";
        })
        .catch(err => {
          this.msg = err.message;
          if (err.status !== 422) {
            Sentry.captureException(err);
          }
        });
    },
    onPhoneInputUpdate(phoneInputInfo) {
      this.phoneInputInfo = phoneInputInfo;
    }
  }
};
</script>

<style lang="scss" scoped>
.uc-form-body {
  @include child-spacing(top, 25px);
}

.phone-input {
  margin: 10px 0 2px;
}

.step-title {
  font-weight: bold;
  text-align: left;
}

.step-errors {
  color: #bf0000;
  font-size: 14px;
  text-align: left;
}

.d-none {
  display: none !important;
}
</style>
