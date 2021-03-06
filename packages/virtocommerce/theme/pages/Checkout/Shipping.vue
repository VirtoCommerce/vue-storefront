<template>
  <div>
    <SfHeading
      :level="3"
      title="Shipping"
      class="sf-heading--left sf-heading--no-underline title"
    />
    <ValidationObserver v-slot="{ handleSubmit, dirty, reset }">
      <form @submit.prevent="handleSubmit(canContinueToPayment(dirty) ? handleShippingMethodSubmit(reset) : handleShippingAddressSubmit(reset))">
        <UserShippingAddresses
          v-if="isAuthenticated && shippingAddresses && shippingAddresses.length"
          :setAsDefault="setAsDefault"
          :shippingAddresses="shippingAddresses"
          :currentAddressId="currentAddressId"
          @setCurrentAddress="setCurrentAddress($event)"
          @changeSetAsDefault="setAsDefault = $event"
        />
        <div class="form" v-if="canAddNewAddress">
          <ValidationProvider name="firstName" rules="required|min:2" v-slot="{ errors }" slim>
            <SfInput
              :value="shippingDetails.firstName"
              @input="firstName => setShippingDetailsAndUnpickAddress({ firstName })"
              label="First name"
              name="firstName"
              class="form__element form__element--half"
              required
              :valid="!errors[0]"
              :errorMessage="errors[0]"
            />
          </ValidationProvider>
          <ValidationProvider name="lastName" rules="required|min:2" v-slot="{ errors }" slim>
            <SfInput
              :value="shippingDetails.lastName"
              @input="lastName => setShippingDetailsAndUnpickAddress({ lastName })"
              label="Last name"
              name="lastName"
              class="form__element form__element--half form__element--half-even"
              required
              :valid="!errors[0]"
              :errorMessage="errors[0]"
            />
          </ValidationProvider>
          <ValidationProvider name="streetName" rules="required|min:2" v-slot="{ errors }" slim>
            <SfInput
              :value="shippingDetails.streetName"
              @input="streetName => setShippingDetailsAndUnpickAddress({ streetName })"
              label="Street name"
              name="streetName"
              class="form__element form__element--half"
              required
              :valid="!errors[0]"
              :errorMessage="errors[0]"
            />
          </ValidationProvider>
          <ValidationProvider name="apartment" rules="required|min:2" v-slot="{ errors }" slim>
            <SfInput
              :value="shippingDetails.streetNumber"
              @input="streetNumber => setShippingDetailsAndUnpickAddress({ streetNumber })"
              label="House/Apartment number"
              name="apartment"
              class="form__element form__element--half form__element--half-even"
              required
              :valid="!errors[0]"
              :errorMessage="errors[0]"
            />
          </ValidationProvider>
          <ValidationProvider name="city" rules="required|min:2" v-slot="{ errors }" slim>
            <SfInput
              :value="shippingDetails.city"
              @input="city => setShippingDetailsAndUnpickAddress({ city })"
              label="City"
              name="city"
              class="form__element form__element--half"
              required
              :valid="!errors[0]"
              :errorMessage="errors[0]"
            />
          </ValidationProvider>
          <ValidationProvider name="zipCode" rules="required|min:2" v-slot="{ errors }" slim>
            <SfInput
              :value="shippingDetails.postalCode"
              @input="postalCode => setShippingDetailsAndUnpickAddress({ postalCode })"
              label="Zip-code"
              name="zipCode"
              class="form__element form__element--half form__element--half-even"
              required
              :valid="!errors[0]"
              :errorMessage="errors[0]"
            />
          </ValidationProvider>
          <ValidationProvider name="country" rules="required|min:2" v-slot="{ errors }" slim>
          <SfSelect
            :selected="shippingDetails.country"
            @input="country => setShippingDetailsAndUnpickAddress({ country })"
            label="Country"
            name="country"
            class="form__element form__element--half form__select sf-select--underlined"
            required
            :valid="!errors[0]"
            :errorMessage="errors[0]"
          >
            <SfSelectOption
              v-for="countryOption in countries"
              :key="countryOption.name"
              :value="countryOption.name"
            >
              {{ countryOption.label }}
            </SfSelectOption>
          </SfSelect>
          </ValidationProvider>
          <ValidationProvider name="phone" rules="required|digits:9" v-slot="{ errors }" slim>
          <SfInput
            :value="shippingDetails.contactInfo.phone"
            @input="phone => setShippingDetailsAndUnpickAddress({ contactInfo: { phone } })"
            label="Phone number"
            name="phone"
            class="form__element form__element--half form__element--half-even"
            required
            :valid="!errors[0]"
            :errorMessage="errors[0]"
          />
          </ValidationProvider>
        </div>
        <SfButton
          v-if="!canAddNewAddress"
          class="color-light form__action-button form__action-button--add-address"
          type="submit"
          @click.native="canAddNewAddress = true"
        >
          Add new address
        </SfButton>
        <SfHeading
          v-if="canContinueToPayment(dirty)"
          :level="3"
          title="Shipping method"
          class="sf-heading--left sf-heading--no-underline title"
        />
        <div class="form">
          <div class="form__radio-group" v-if="canContinueToPayment(dirty)">
            <SfRadio
              v-for="item in shippingMethods"
              :key="checkoutGetters.getShippingMethodName(item)"
              :label="checkoutGetters.getShippingMethodName(item)"
              :value="checkoutGetters.getShippingMethodId(item)"
              :selected="checkoutGetters.getShippingMethodId(chosenShippingMethod)"
              @input="setShippingMethod(item, { save: true })"
              name="shippingMethod"
              :description="checkoutGetters.getShippingMethodDescription(item)"
              class="form__radio shipping"
            >
              <template #label="{label}">
                <div class="sf-radio__label shipping__label">
                  <div>{{ label }}</div>
                  <div>${{ checkoutGetters.getShippingMethodPrice(item) }}</div>
                </div>
              </template>
              <template #description="{description}">
                <div class="sf-radio__description shipping__description">
                  <div class="shipping__info">
                    {{ description }}
                  </div>
                </div>
              </template>
            </SfRadio>
          </div>
          <div class="form__action">
            <nuxt-link to="/checkout/personal-details" class="sf-button color-secondary form__back-button">Go back</nuxt-link>
            <SfButton class="form__action-button" type="submit" v-if="canContinueToPayment(dirty)" :disabled="!isShippingMethodCompleted || loading.shippingAddress">
              Continue to payment
            </SfButton>
            <SfButton class="form__action-button" type="submit" :disabled="loading.shippingMethods" v-else>
              Select shipping method
            </SfButton>
          </div>
        </div>
      </form>
    </ValidationObserver>
  </div>
</template>

<script>
import {
  SfHeading,
  SfInput,
  SfButton,
  SfSelect,
  SfRadio,
  SfCheckbox
} from '@storefront-ui/vue';

import { useCheckout, useUserShipping, useUser, checkoutGetters, userShippingGetters } from '@vue-storefront/virtocommerce';
import { ValidationProvider, ValidationObserver, extend } from 'vee-validate';
import { required, min, digits } from 'vee-validate/dist/rules';
import { onSSR } from '@vue-storefront/core';
import { ref, onMounted, computed } from '@vue/composition-api';

extend('required', {
  ...required,
  message: 'This field is required'
});
extend('min', {
  ...min,
  message: 'The field should have at least {length} characters'
});
extend('digits', {
  ...digits,
  message: 'Please provide a valid phone number'
});

const COUNTRIES = [
  { key: 'US',
    label: 'United States' },
  { key: 'UK',
    label: 'United Kingdom' },
  { key: 'IT',
    label: 'Italy' },
  { key: 'PL',
    label: 'Poland' }
];

export default {
  name: 'Shipping',
  components: {
    SfHeading,
    SfInput,
    SfButton,
    SfSelect,
    SfRadio,
    SfCheckbox,
    UserShippingAddresses: () => import('~/components/Checkout/UserShippingAddresses'),
    ValidationProvider,
    ValidationObserver
  },
  setup(props, context) {
    const {
      shippingDetails,
      chosenShippingMethod,
      setShippingDetails,
      setShippingMethod,
      shippingMethods,
      loadShippingMethods,
      isShippingAddressCompleted,
      isShippingMethodCompleted,
      loadDetails,
      loading
    } = useCheckout();
    const { shipping, load: loadShipping, setDefault } = useUserShipping();
    const { isAuthenticated } = useUser();

    const canAddNewAddress = ref(true);
    const addressIsModified = ref(false);
    const currentAddressId = ref(-1);
    const setAsDefault = ref(false);

    const mapAbstractAddressToIntegrationAddress = address => ({
      ...shippingDetails.value,
      contactInfo: {
        ...shippingDetails.value.contactInfo,
        phone: address.phoneNumber
      },
      streetNumber: address.apartment,
      city: address.city,
      country: address.country,
      state: address.state,
      firstName: address.firstName,
      lastName: address.lastName,
      streetName: address.streetName,
      postalCode: address.zipCode
    });

    const setCurrentAddress = async (addressId) => {
      const chosenAddress = userShippingGetters.getAddresses(shipping.value, { id: addressId });
      if (!chosenAddress || !chosenAddress.length) {
        return;
      }
      currentAddressId.value = addressId;
      await setShippingDetails(mapAbstractAddressToIntegrationAddress(chosenAddress[0]));
      addressIsModified.value = true;
    };

    onSSR(async () => {
      await loadDetails();
      await loadShippingMethods();
    });

    onMounted(async () => {
      if (isAuthenticated.value) {
        await loadShipping();
        const shippingAddresses = userShippingGetters.getAddresses(shipping.value);
        if (!shippingAddresses || !shippingAddresses.length) {
          return;
        }
        canAddNewAddress.value = false;
        if (shippingAddresses[0].isDefault) {
          setCurrentAddress(shippingAddresses[0].id);
        }
      }
    });

    const handleShippingAddressSubmit = (reset) => async () => {
      await setShippingDetails(shippingDetails.value, { save: true });
      await loadShippingMethods();
      reset();
      if (currentAddressId.value > -1 && setAsDefault.value) {
        const chosenAddress = userShippingGetters.getAddresses(shipping.value, { id: currentAddressId.value });
        if (!chosenAddress || !chosenAddress.length) {
          return;
        }
        await setDefault(chosenAddress[0]);
      }
      addressIsModified.value = false;
    };
    const handleShippingMethodSubmit = (reset) => async () => {
      await setShippingMethod(chosenShippingMethod.value, { save: true });
      reset();
      context.root.$router.push('/checkout/payment');
    };

    const setShippingDetailsAndUnpickAddress = value => {
      setShippingDetails(value);
      currentAddressId.value = -1;
      addressIsModified.value = true;
    };

    const canContinueToPayment = dirty => isShippingAddressCompleted.value && !dirty && !addressIsModified.value;

    return {
      loading,
      handleShippingAddressSubmit,
      handleShippingMethodSubmit,
      isShippingAddressCompleted,
      isShippingMethodCompleted,
      setShippingDetailsAndUnpickAddress,
      setShippingMethod,
      shippingDetails,
      chosenShippingMethod,
      shippingMethods,
      checkoutGetters,
      countries: COUNTRIES,
      shippingAddresses: computed(() => userShippingGetters.getAddresses(shipping.value)),
      canAddNewAddress,
      addressIsModified,
      isAuthenticated,
      currentAddressId: computed(() => currentAddressId.value),
      setAsDefault,
      setCurrentAddress,
      canContinueToPayment
    };
  }
};
</script>

<style lang="scss" scoped>

.title {
  margin: var(--spacer-xl) 0 var(--spacer-base) 0;
}
.form {
  --button-width: 100%;
  &__select {
    display: flex;
    align-items: center;
    --select-option-font-size: var(--font-size--lg);
    ::v-deep .sf-select__dropdown {
      font-size: var(--font-size--lg);
      margin: 0;
      color: var(--c-text);
      font-family: var(--font-family--secondary);
      font-weight: var(--font-weight--normal);
    }
  }
  @include for-desktop {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    --button-width: auto;
  }
  &__element {
    margin: 0 0 var(--spacer-xl) 0;
    @include for-desktop {
      flex: 0 0 100%;
    }
    &--half {
      @include for-desktop {
        flex: 1 1 50%;
      }
      &-even {
        @include for-desktop {
          padding: 0 0 0 var(--spacer-xl);
        }
      }
    }
  }
  &__group {
    display: flex;
    align-items: center;
  }
  &__action {
    @include for-desktop {
      flex: 0 0 100%;
      display: flex;
    }
  }
  &__action-button {
    &--secondary {
      @include for-desktop {
        order: -1;
        text-align: left;
      }
    }
    &--add-address {
      width: 100%;
      margin: 0;
      @include for-desktop {
        margin: 0 0 var(--spacer-lg) 0;
        width: auto;
      }
    }
  }
  &__back-button {
    margin: var(--spacer-xl) 0 var(--spacer-sm);
    &:hover {
      color:  var(--c-white);
    }
    @include for-desktop {
      margin: 0 var(--spacer-xl) 0 0;
    }
  }
  &__radio-group {
    flex: 0 0 100%;
    margin: 0 0 var(--spacer-xl) 0;
    @include for-desktop {
      margin: 0 0 var(--spacer-2xl) 0;
    }

  }
}
.shipping {
  &__label {
    display: flex;
    justify-content: space-between;
  }
  &__description {
    --radio-description-margin: 0;
    --radio-description-font-size: var(--font-xs);
  }
}
</style>
