<template>
  <modal :show.sync="show" :on-close="onClose">
    <div class="column content-box">
      <woot-modal-header
        :header-title="pageTitle"
      />
      <form class="row medium-8" v-on:submit.prevent="editCannedResponse()">
        <div class="medium-12 columns">
          <label :class="{ 'error': $v.shortCode.$error }">
            {{ $t('CANNED_MGMT.EDIT.FORM.SHORT_CODE.LABEL') }}
            <input type="text" v-model.trim="shortCode" @input="$v.shortCode.$touch" :placeholder="$t('CANNED_MGMT.EDIT.FORM.SHORT_CODE.PLACEHOLDER')">
          </label>
        </div>

        <div class="medium-12 columns">
          <label :class="{ 'error': $v.content.$error }">
            {{ $t('CANNED_MGMT.EDIT.FORM.CONTENT.LABEL') }}
            <input type="text" v-model.trim="content" @input="$v.content.$touch" :placeholder="$t('CANNED_MGMT.EDIT.FORM.CONTENT.PLACEHOLDER')">
          </label>
        </div>
        <div class="modal-footer">
          <div class="medium-12 columns">
            <woot-submit-button
              :disabled="$v.content.$invalid || $v.shortCode.$invalid || editCanned.showLoading"
              :button-text="$t('CANNED_MGMT.EDIT.FORM.SUBMIT')"
              :loading="editCanned.showLoading"
            />
            <a @click="onClose">Cancel</a>
          </div>
        </div>
      </form>
    </div>
  </modal>
</template>

<script>
/* global bus */
/* eslint no-console: 0 */
import { required, minLength } from 'vuelidate/lib/validators';

import PageHeader from '../SettingsSubPageHeader';
import WootSubmitButton from '../../../../components/buttons/FormSubmitButton';
import Modal from '../../../../components/Modal';

export default {
  components: {
    PageHeader,
    WootSubmitButton,
    Modal,
  },
  props: {
    id: Number,
    edcontent: String,
    edshortCode: String,
    onClose: Function,
  },
  data() {
    return {
      editCanned: {
        showAlert: false,
        showLoading: false,
        message: '',
      },
      shortCode: this.edshortCode,
      content: this.edcontent,
      show: true,
    };
  },
  validations: {
    shortCode: {
      required,
      minLength: minLength(2),
    },
    content: {
      required,
    },
  },
  computed: {
    pageTitle() {
      return `${this.$t('CANNED_MGMT.EDIT.TITLE')} - ${this.edshortCode}`;
    },
  },
  methods: {
    setPageName({ name }) {
      this.$v.content.$touch();
      this.content = name;
    },
    showAlert() {
      bus.$emit('newToastMessage', this.editCanned.message);
    },
    resetForm() {
      this.shortCode = this.content = '';
      this.$v.shortCode.$reset();
      this.$v.content.$reset();
    },
    editCannedResponse() {
      // Show loading on button
      this.editCanned.showLoading = true;
      // Make API Calls
      this.$store.dispatch('editCannedResponse', {
        id: this.id,
        name: this.shortCode,
        content: this.content,
      })
      .then(() => {
        // Reset Form, Show success message
        this.editCanned.showLoading = false;
        this.editCanned.message = this.$t('CANNED_MGMT.EDIT.API.SUCCESS_MESSAGE');
        this.showAlert();
        this.resetForm();
        setTimeout(() => {
          this.onClose();
        }, 10);
      })
      .catch(() => {
        this.editCanned.showLoading = false;
        this.editCanned.message = this.$t('CANNED_MGMT.EDIT.API.ERROR_MESSAGE');
        this.showAlert();
      });
    },
  },
};
</script>
