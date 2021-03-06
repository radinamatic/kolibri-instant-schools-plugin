<template>

  <div id="signup-page">

    <ui-toolbar type="colored" textColor="white">
      <template slot="icon">
        <img src="../img/instant-school-logo.png" class="app-bar-icon" alt="">
      </template>
      <template slot="brand">
        {{ $tr('appBarHeader') }}
      </template>
      <div slot="actions">
        <router-link id="signin" :to="signInPage">
          <span>{{ $tr('logIn') }}</span>
        </router-link>
      </div>
    </ui-toolbar>

    <form class="signup-form" ref="form" @submit.prevent="signUp">
      <ui-alert type="error" @dismiss="resetSignUpState" v-if="unknownError">
        {{errorMessage}}
      </ui-alert>

      <h1 class="signup-title">{{ $tr('createAccount') }}</h1>

      <k-textbox
        ref="name"
        id="name"
        type="text"
        autocomplete="name"
        :label="$tr('name')"
        :maxlength="120"
        :autofocus="true"
        :invalid="nameIsInvalid"
        :invalidText="nameIsInvalidText"
        @blur="nameBlurred = true"
        v-model="name"
      />

      <k-textbox
        ref="username"
        id="username"
        type="tel"
        autocomplete="tel"
        :label="$tr('phoneNumberLabel')"
        :invalid="usernameIsInvalid"
        :invalidText="usernameIsInvalidText"
        @blur="usernameBlurred = true"
        @input="resetSignUpState"
        v-model="username"
      />

      <k-textbox
        ref="password"
        id="password"
        type="password"
        autocomplete="new-password"
        :label="$tr('password')"
        :invalid="passwordIsInvalid"
        :invalidText="passwordIsInvalidText"
        @blur="passwordBlurred = true"
        v-model="password"
      />

      <k-textbox
        ref="confirmedPassword"
        id="confirmed-password"
        type="password"
        autocomplete="new-password"
        :label="$tr('reEnterPassword')"
        :invalid="confirmedPasswordIsInvalid"
        :invalidText="confirmedPasswordIsInvalidText"
        @blur="confirmedPasswordBlurred = true"
        v-model="confirmedPassword"
      />

      <div class="terms-agreement">
        <label for="terms-agreement-checkbox">
          <button
            type="button"
            class="terms-agreement-view-prompt"
            @click="showTerms = true"
          >
            {{ $tr('viewTermsOfServicePrompt') }}
          </button>
        </label>

        <k-checkbox
          :class="['terms-agreement-checkbox', termsNotAgreed ? 'invalid' : '']"
          id="terms-agreement-checkbox"
          :checked="termsAgreed"
          @change="termsAgreed = $event"
          @blur="termsAgreementCheckboxBlurred = true"
          :label="$tr('termsAgreementLabel')"
        />

        <label
          v-if="termsNotAgreed"
          aria-live="polite"
          for="terms-agreement-checkbox"
          class="terms-agreement-error-box"
        >
          {{ termsNotAgreedText }}
        </label>
      </div>

      <k-button :disabled="busy" :primary="true" :text="$tr('finish')" type="submit" />

    </form>

    <core-modal
      v-if="showTerms"
      @cancel="showTerms = false"
      :title="$tr('termsOfServiceModalHeader')"
    >
      <iframe class="terms" src="/content/databases/about/tos.txt"></iframe>
    </core-modal>

    <div class="footer">
      <language-switcher :footer="true"/>
    </div>
  </div>

</template>


<script>

  import { signUp, resetSignUpState } from '../../state/actions';
  import { PageNames } from '../../constants';
  import { validateUsername } from 'kolibri.utils.validators';
  import kButton from 'kolibri.coreVue.components.kButton';
  import uiAlert from 'keen-ui/src/UiAlert';
  import kCheckbox from 'kolibri.coreVue.components.kCheckbox';
  import coreModal from 'kolibri.coreVue.components.coreModal';
  import kTextbox from 'kolibri.coreVue.components.kTextbox';
  import uiToolbar from 'keen-ui/src/UiToolbar';
  import languageSwitcher from 'kolibri.coreVue.components.languageSwitcher';

  export default {
    name: 'signUpPage',
    $trs: {
      createAccount: 'Create an account',
      name: 'Full name',
      phoneNumberLabel: 'Phone Number',
      password: 'Password',
      reEnterPassword: 'Re-enter password',
      passwordMatchError: 'Passwords do not match',
      genericError: 'Something went wrong during sign up!',
      phoneNumberInvalid: 'A valid phone number has at least 9 digits',
      accountAlreadyExistsError: 'An account with that phone number already exists',
      logIn: 'Sign in',
      termsAgreementLabel: 'I agree to the terms of service & privacy policy',
      termsOfServiceModalHeader: 'Terms of service & privacy policy',
      viewTermsOfServicePrompt: 'View terms of service & privacy policy',
      appBarHeader: 'Instant Schools',
      finish: 'Finish',
      required: 'This field is required',
    },
    components: {
      kButton,
      kTextbox,
      uiToolbar,
      languageSwitcher,
      coreModal,
      kCheckbox,
      uiAlert,
    },
    data: () => ({
      name: '',
      username: '',
      password: '',
      confirmedPassword: '',
      nameBlurred: false,
      usernameBlurred: false,
      passwordBlurred: false,
      confirmedPasswordBlurred: false,
      formSubmitted: false,
      showTerms: false,
      termsAgreed: false,
      termsAgreementCheckboxBlurred: false,
    }),
    computed: {
      signInPage() {
        return { name: PageNames.SIGN_IN };
      },
      facilityList() {
        return this.facilities.map(facility => ({
          label: facility.name,
          id: facility.id,
        }));
      },
      nameIsInvalidText() {
        if (this.nameBlurred || this.formSubmitted) {
          if (this.name === '') {
            return this.$tr('required');
          }
        }
        return '';
      },
      nameIsInvalid() {
        return !!this.nameIsInvalidText;
      },
      usernameDoesNotExistYet() {
        if (this.errorCode === 400) {
          return false;
        }
        return true;
      },
      usernameIsInvalidText() {
        if (this.usernameBlurred || this.formSubmitted) {
          if (this.username === '') {
            return this.$tr('required');
          }
          if (!this.validatePhoneNumber(this.username)) {
            return this.$tr('phoneNumberInvalid');
          }
          if (!this.usernameDoesNotExistYet) {
            return this.$tr('accountAlreadyExistsError');
          }
        }
        return '';
      },
      usernameIsInvalid() {
        return !!this.usernameIsInvalidText;
      },
      passwordIsInvalidText() {
        if (this.passwordBlurred || this.formSubmitted) {
          if (this.password === '') {
            return this.$tr('required');
          }
        }
        return '';
      },
      passwordIsInvalid() {
        return !!this.passwordIsInvalidText;
      },
      confirmedPasswordIsInvalidText() {
        if (this.confirmedPasswordBlurred || this.formSubmitted) {
          if (this.confirmedPassword === '') {
            return this.$tr('required');
          }
          if (this.confirmedPassword !== this.password) {
            return this.$tr('passwordMatchError');
          }
        }
        return '';
      },
      confirmedPasswordIsInvalid() {
        return !!this.confirmedPasswordIsInvalidText;
      },
      termsNotAgreedText() {
        if ((this.termsAgreementCheckboxBlurred || this.formSubmitted) && !this.termsAgreed) {
          return this.$tr('required');
        }
        return '';
      },
      termsNotAgreed() {
        return !!this.termsNotAgreedText;
      },
      formIsValid() {
        return (
          !this.nameIsInvalid &&
          !this.usernameIsInvalid &&
          !this.passwordIsInvalid &&
          !this.confirmedPasswordIsInvalid &&
          !this.termsNotAgreed
        );
      },
      unknownError() {
        if (this.errorCode) {
          return this.errorCode !== 400;
        }
        return false;
      },
      errorMessage() {
        return this.backendErrorMessage || this.$tr('genericError');
      },
    },
    methods: {
      signUp() {
        this.formSubmitted = true;
        const canSubmit = this.formIsValid && !this.busy;
        if (canSubmit) {
          this.signUpAction({
            facility: this.facilityList[0],
            full_name: this.name,
            username: this.username,
            password: this.password,
          });
        } else {
          this.focusOnInvalidField();
        }
      },
      focusOnInvalidField() {
        if (this.nameIsInvalid) {
          this.$refs.name.focus();
        } else if (this.usernameIsInvalid) {
          this.$refs.username.focus();
        } else if (this.passwordIsInvalid) {
          this.$refs.password.focus();
        } else if (this.confirmedPasswordIsInvalid) {
          this.$refs.confirmedPassword.focus();
        }
      },
      validatePhoneNumber() {
        const strippedPhoneNumber = this.username.replace(/\D/g, '');
        return strippedPhoneNumber.length > 8;
      },
    },
    vuex: {
      getters: {
        session: state => state.core.session,
        errorCode: state => state.pageState.errorCode,
        busy: state => state.pageState.busy,
        backendErrorMessage: state => state.pageState.errorMessage,
        facilities: state => state.core.facilities,
      },
      actions: {
        signUpAction: signUp,
        resetSignUpState: resetSignUpState,
      },
    },
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.definitions'

  $iphone-5-width = 320px
  $vertical-page-margin = 100px
  $logo-size = 36px
  $logo-margin = 16px
  $keen-invalid-md-red = #f44336
  $form-item-spacing = 16px // defined in k-textbox

  // component, highest level
  #signup-page
    width: 100%
    height: 100%

  // Action Bar
  #logo
    // 1.63 * font height
    height: $logo-size
    display: inline-block
    margin-left: $logo-margin

  #signin
    margin-right: 1em
    color: white
    text-decoration: none

  // Form
  .signup-title
    text-align: center

  .signup-form
    margin-top: $vertical-page-margin
    margin-left: auto
    margin-right: auto
    width: ($iphone-5-width - 20)px
    max-width: 100%

  .terms
    height: 80vh
    width: 80vw
    border: none
    &-agreement
      $height-of-prompt = 18px + 16px
      $height-of-checkbox = 48px + 16px
      $k-textbox-text-distance = 24px // distance from top of text to its label container
      $height-of-error = 16px

      display: inline-block
      height: $height-of-prompt + $height-of-checkbox + $height-of-error
      margin-bottom: $form-item-spacing // margin defined for k-textbox
      margin-top: $k-textbox-text-distance
      max-width: 100%
      &-view-prompt

        // duplicating styles from `<a>` in core theme
        color: $core-action-normal
        transition: color $core-time ease-out
        max-width: 100%
        white-space: nowrap
        overflow: hidden
        text-overflow: ellipsis
        &:hover
          color: $core-action-dark
        &:hover:focus, &:focus
          outline: $core-outline
        // end dupe

        display: block
        margin-bottom: $form-item-spacing
        padding: 0
        text-decoration: underline
      &-checkbox
        margin-top: 0
        margin-bottom: $form-item-spacing
        &.invalid
          color: $keen-invalid-md-red
      &-error-box
        display: block
        color: $keen-invalid-md-red // same color as input error messages
        font-size: 14px // same as error messages from inputs

  .app-bar-icon
    display: inline-block
    margin-left: $logo-margin
    height: $logo-size
    width: $logo-size

  .footer
    margin: 36px
    margin-top: 96px

</style>
