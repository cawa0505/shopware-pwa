<template>
  <div class="sw-password">
    <slot>
      <slot name="message" :user="user">
        <p class="message">
          If you want to change the password to access your account, enter the
          following information:
        </p>
      </slot>

      <SfAlert
        v-for="(message, index) in userErrorMessages"
        :key="index"
        class="sw-password__alert"
        type="danger"
        :message="message"
      />

      <div class="sw-password__form form">
        <slot name="form">
          <div class="form">
            <SwInput
              v-model="password"
              :valid="!$v.password.$error"
              error-message="Current password is required"
              type="password"
              name="currentPassword"
              label="Current Password"
              class="form__element"
              required
              @blur="$v.password.$touch()"
            />
            <SwInput
              v-model="newPassword"
              :valid="!$v.newPassword.$error"
              error-message="This field is required"
              type="password"
              name="newPassword"
              label="New Password"
              class="form__element form__element--half"
              required
              @blur="$v.newPassword.$touch()"
            />
            <SwInput
              v-model="newPasswordConfirm"
              :valid="!$v.newPasswordConfirm.$error"
              error-message="This filed must be same as new password"
              type="password"
              name="repeatPassword"
              label="Repeat Password"
              required
              class="form__element form__element--half form__element--half-even"
              @blur="$v.newPasswordConfirm.$touch()"
            />
            <SwButton
              class="form__button"
              :disabled="$v.$invalid"
              @click="invokeUpdate"
            >
              Update password
            </SwButton>
          </div>
        </slot>
      </div>
    </slot>
  </div>
</template>

<script>
import { validationMixin } from "vuelidate"
import { required, minLength, sameAs } from "vuelidate/lib/validators"
import { computed } from "@vue/composition-api"
import { SfAlert } from "@storefront-ui/vue"
import { useUser } from "@shopware-pwa/composables"
import { getMessagesFromErrorsArray } from "@shopware-pwa/helpers"
import SwButton from "@shopware-pwa/default-theme/components/atoms/SwButton"
import SwInput from "@shopware-pwa/default-theme/components/atoms/SwInput"

export default {
  name: "SwPassword",
  components: { SwInput, SwButton, SfAlert },
  mixins: [validationMixin],
  props: {},
  setup(props, {root}) {
    const { user, error: userError, updatePassword, refreshUser } = useUser(root)
    const userErrorMessages = computed(() =>
      getMessagesFromErrorsArray(userError.value && userError.value.message)
    )

    return {
      refreshUser,
      updatePassword,
      user,
      userErrorMessages,
    }
  },
  data() {
    return {
      password: "",
      newPassword: "",
      newPasswordConfirm: "",
      email: this.user && this.user.email,
    }
  },
  methods: {
    async invokeUpdate() {
      const passwordChanged = await this.updatePassword({
        password: this.password,
        newPassword: this.newPassword,
        newPasswordConfirm: this.newPasswordConfirm,
      })
      await this.refreshUser()
    },
  },
  validations: {
    password: {
      required,
      minLenght: minLength(8),
    },
    newPassword: {
      required,
      minLength: minLength(8),
    },
    newPasswordConfirm: {
      required,
      sameAsNewPassword: sameAs("newPassword"),
    },
  },
}
</script>

<style lang="scss" scoped>
@import "@/assets/scss/variables";

.sw-password {
  &__alert {
    margin-bottom: var(--spacer-base);
  }
}

.form {
  @include for-desktop {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
  }
  &__element {
    margin: 0 0 var(--spacer-lg) 0;
    @include for-desktop {
      flex: 0 0 100%;
    }
    &--half {
      @include for-desktop {
        flex: 1 1 50%;
      }
      &-even {
        @include for-desktop {
          padding: 0 0 0 var(--spacer-lg);
        }
      }
    }
  }
  &__button {
    --button-width: 100%;
    @include for-desktop {
      --button-width: auto;
    }
  }
}

.message {
  margin: 0 0 var(--spacer-xl) 0;
  color: var(--c-dark-variant);
  &__label {
    font-weight: 400;
  }
  &--second {
    padding: 4rem;
  }
}
</style>
