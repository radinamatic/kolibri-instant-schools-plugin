<template>

  <div>
    <core-base :navBarNeeded="navBarNeeded" :topLevelPageName="topLevelPageName" :appBarTitle="appBarTitle">
      <component v-if="navBarNeeded" :is="currentPage"/>
    </core-base>
    <div v-if="!navBarNeeded" class="full-page">
      <component :is="currentPage"/>
    </div>
  </div>

</template>


<script>

  import store from '../state/store';
  import { PageNames } from '../constants';
  import { TopLevelPageNames } from 'kolibri.coreVue.vuex.constants';
  import coreBase from 'kolibri.coreVue.components.coreBase';
  import signInPage from './sign-in-page';
  import signUpPage from './sign-up-page';
  import accountPage from './account-page';
  import selectProfilePage from './select-profile-page';
  import resetPasswordPage from './reset-password-page';

  export default {
    $trs: { userAccountTitle: 'Account' },
    name: 'userPlugin',
    components: {
      coreBase,
      selectProfilePage,
      resetPasswordPage,
      signInPage,
      signUpPage,
      accountPage,
    },
    computed: {
      appBarTitle() {
        if (this.pageName === PageNames.ACCOUNT) {
          return this.$tr('userAccountTitle');
        }
        return '';
      },
      topLevelPageName: () => TopLevelPageNames.USER,
      currentPage() {
        if (this.pageName === PageNames.SIGN_IN) {
          return 'sign-in-page';
        }
        if (this.pageName === PageNames.SIGN_UP) {
          return 'sign-up-page';
        }
        if (this.pageName === PageNames.ACCOUNT) {
          return 'account-page';
        }
        if (this.pageName === PageNames.SELECT_PROFILE) {
          return 'selectProfilePage';
        }
        if (this.pageName === PageNames.RESET_PASSWORD) {
          return 'resetPasswordPage';
        }
        return null;
      },
      navBarNeeded() {
        if (this.pageName === PageNames.SIGN_IN) {
          return false;
        }
        if (this.pageName === PageNames.SIGN_UP) {
          return false;
        }
        if (this.pageName === PageNames.SELECT_PROFILE) {
          return false;
        }
        return true;
      },
    },
    vuex: { getters: { pageName: state => state.pageName } },
    store,
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.definitions'

  .full-page
    position: absolute
    top: 0
    height: 100%
    width: 100%

</style>
