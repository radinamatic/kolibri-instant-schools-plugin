<template>

  <div>
    <div class="message">
      {{ message }}
    </div>
    <div class="buttons">
      <k-button
        :text="$tr('close')"
        @click="$emit('close')"
        :primary="true"
      />
    </div>
  </div>

</template>


<script>

  import kButton from 'kolibri.coreVue.components.kButton';
  import { RequestTokenStates as STATES } from '../../constants';

  export default {
    name: 'resetPasswordModalStatus',
    components: {
      kButton,
    },
    props: {
      status: {
        type: String,
        required: true,
      },
    },
    computed: {
      message() {
        switch (this.status) {
          case STATES.MESSAGE_SENT:
            return this.$tr('messageSent');
          case STATES.SMS_SERVICE_ERROR:
            return this.$tr('smsServiceUnavailable');
          default:
            return '';
        }
      },
    },
    $trs: {
      close: 'Close',
      goBack: 'Go back',
      messageSent:
        'A text message (SMS) has been sent to your phone with instructions to reset your password.',
      smsServiceUnavailable: 'The text message (SMS) service is not currently available',
    },
  };

</script>


<style lang="stylus" scoped>

  .message
    text-align: left

  .buttons
    text-align: right
    margin-top: 1em

  button
    margin-right: 0

</style>
