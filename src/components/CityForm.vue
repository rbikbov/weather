<template>
  <CardWrapper class="city-form-wrapper">
    <div class="city-form-wrapper__title">Choose a city</div>
    <div class="city-form-wrapper__subtitle">
      Type city name
      <!-- To find city start typing and pick one from the suggestions -->
    </div>
    <form class="city-form" @submit.prevent="onSubmit">
      <BaseInput
        v-model.trim="$v.formData.cityName.$model"
        placeholder="Search city"
        :error="cityNameError || error"
        :autofocus="true"
        @blur="$v.formData.cityName.$touch"
      />
      <div class="city-form-actions">
        <BaseButton
          type="button"
          class="city-form__btn city-form__btn_clear"
          :disabled="!formData.cityName"
          @click="onClear"
        >
          CLEAR
        </BaseButton>
        <BaseButton
          type="button"
          class="city-form__btn city-form__btn_cancel"
          @click="onCancel"
        >
          CANCEL
        </BaseButton>
        <BaseButton
          type="submit"
          :loading="loading"
          :disabled="isAddButtonDisabled"
        >
          ADD
        </BaseButton>
      </div>
    </form>
  </CardWrapper>
</template>

<script>
import { required, helpers } from 'vuelidate/lib/validators'
import BaseButton from './BaseButton.vue'
import BaseInput from './BaseInput.vue'
import CardWrapper from './CardWrapper.vue'

const cityNameRegex = helpers.regex('regex', /^[a-zA-Z -']*$/)

export default {
  name: 'CityForm',
  components: {
    BaseInput,
    BaseButton,
    CardWrapper
  },
  props: {
    loading: {
      type: Boolean
    },
    error: {
      type: String
    },
    addedCities: {
      type: Array,
      default: () => []
    }
  },
  data () {
    return {
      formData: {
        cityName: ''
      }
    }
  },
  validations: {
    formData: {
      cityName: {
        required,
        regex: cityNameRegex,
        isUnique (value) {
          if (value === '') return true

          return !this.addedCities.some(cn => cn.toLowerCase() === value.toLowerCase())
        }
      }
    }
  },
  computed: {
    cityNameError () {
      if (!this.$v.formData.cityName.$error) {
        return ''
      }
      if (!this.$v.formData.cityName.required) {
        return 'City name is required'
      }
      if (!this.$v.formData.cityName.regex) {
        return 'City name can contain only English letters'
      }
      if (!this.$v.formData.cityName.isUnique) {
        return 'This city already in added'
      }
      return 'Unknown error'
    },
    isAddButtonDisabled () {
      return this.loading || Boolean(this.cityNameError)
    }
  },
  methods: {
    onClear () {
      this.formData.cityName = ''
    },
    onCancel () {
      this.$emit('cancel')
    },
    onSubmit () {
      this.$v.$touch()
      if (this.$v.$invalid) {
        return
      }
      this.$emit('add', this.formData)
    }
  },
  watch: {
    formData: {
      deep: true,
      handler (newVal) {
        this.$emit('formChanged', newVal)
      }
    }
  }
}
</script>

<style lang="scss">
.city-form-wrapper {
  background: #fff;

  @include breakpoint(xs) {
    min-width: calc(100% - 20px);
  }

  @include breakpoint(sm) {
    min-width: 451px;
  }

  @include breakpoint(md) {
    min-width: 551px;
  }

  @include breakpoint(xl) {
    min-width: 751px;
  }

  &__title {
    margin: 0 0 16px 0;
    font-size: 32px;
    font-style: normal;
    font-weight: bold;
    line-height: 38px;
    text-align: left;
  }

  &__subtitle {
    margin: 0 0 60px 0;
    font-size: 18px;
    font-style: normal;
    font-weight: normal;
    line-height: 24px;
    text-align: left;
  }
}

.city-form {
  &-actions {
    display: flex;
    margin-top: 140px;
  }

  &__btn {
    &_clear {
      margin-right: auto;
    }

    &_cancel {
      margin-right: 31px;
    }
  }
}
</style>
