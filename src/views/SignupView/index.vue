<template>
  <form-page-template>
    <div class="uc-form">
      <div class="uc-form-header">
        <router-link to="/login" class="uc-form-header-link"
          >Log In</router-link
        >
        <div class="uc-form-header-link--active">Sign Up</div>
      </div>

      <volunteer-form v-if="this.userSelection === 'volunteer'" />
      <student-form v-else-if="this.userSelection === 'student'" />
      <div v-else class="uc-form-body">
        <b>Are you a student or a volunteer?</b>

        <button
          class="uc-form-button"
          type="submit"
          @click.prevent="selectStudent()"
        >
          Student
        </button>

        <button
          class="uc-form-button"
          type="submit"
          @click.prevent="selectVolunteer()"
        >
          Volunteer
        </button>
      </div>
    </div>
  </form-page-template>
</template>

<script>
import FormPageTemplate from "@/components/FormPageTemplate";
import StudentForm from "./StudentForm";
import VolunteerForm from "./VolunteerForm";

export default {
  name: "signup-view",
  components: {
    FormPageTemplate,
    StudentForm,
    VolunteerForm
  },
  created() {
    this.$store.dispatch("app/hideNavigation");
  },
  data() {
    return {
      userSelection: null
    };
  },
  methods: {
    selectVolunteer() {
      this.userSelection = "volunteer";
    },
    selectStudent() {
      this.userSelection = "student";
    }
  }
};
</script>

<style lang="scss" scoped></style>
