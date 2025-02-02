<template>
  <div :id="inputId">
    <template v-if="$slots['input']">
      <slot
        name="input"
        v-bind="$attrs"
      ></slot>
    </template>
    <input
      v-else-if="!$slots['input']"
      ref="input"
      v-bind="$attrs"
      v-on="$attrs"
    />
  </div>
</template>

<script>
import { bindProps, getPropsValues } from '../utils/bindProps.js'
import downArrowSimulator from '../utils/simulateArrowDown.js'
import { mappedPropsToVueProps } from './build-component'
import { v4 } from 'uuid'

const mappedProps = {
  bounds: {
    type: Object,
  },
  componentRestrictions: {
    type: Object,
    // Do not bind -- must check for undefined
    // in the property
    noBind: true,
  },
  types: {
    type: Array,
    default: function () {
      return []
    },
  },
}

const props = {
  selectFirstOnEnter: {
    required: false,
    type: Boolean,
    default: false,
  },
  options: {
    type: Object,
  },
}

export default {
  data() {
    return {
      inputId: v4()
    }
  },
  mounted() {
    const _this = this;

    this.$gmapApiPromiseLazy().then(() => {
      // get correct input from fallback or slot
      let refInput = _this.$refs.input

      let inputSlot = _this.$slots.input()

      console.log('inputSlot:', inputSlot)

      refInput = document.getElementById(this.inputId).querySelectorAll('input')[0]

      /*
      if (inputSlot) {
        const refName = inputSlot[0].props.ref;
        const scopedInput = inputSlot[0].ref.i.ctx.$refs[refName];
        if (scopedInput) {
          refInput = scopedInput;
          if(refInput.tagName.toLowerCase() !== 'input'){
            const inputs = refInput.querySelectorAll('input')
            if(inputs.length == 0){
              console.error('No input tag found inside ref "'+refName+'"')
            } else {
              refInput = inputs[0]
            }
          } else {
            refInput = scopedInput.$el.querySelectorAll('input')[0]
          }
        } else {
          console.error("Failed to find scopedInput (element with ref='"+refName+"')")
        }
      }
      */
      if (this.selectFirstOnEnter) {
        downArrowSimulator(refInput)
      }

      if (typeof google.maps.places.Autocomplete !== 'function') {
        throw new Error(
          "google.maps.places.Autocomplete is undefined. Did you add 'places' to libraries when loading Google Maps?"
        )
      }

      /* eslint-disable no-unused-vars */
      const finalOptions = {
        ...getPropsValues(this, mappedProps),
        ...this.options,
      }

      this.$autocomplete = new google.maps.places.Autocomplete(refInput, finalOptions)
      bindProps(this, this.$autocomplete, mappedProps)

      this.$watch('componentRestrictions', (v) => {
        if (v !== undefined) {
          this.$autocomplete.setComponentRestrictions(v)
        }
      })

      // Not using `bindEvents` because we also want
      // to return the result of `getPlace()`
      this.$autocomplete.addListener('place_changed', () => {
        this.$emit('place_changed', this.$autocomplete.getPlace())
      })
    })
  },
  props: {
    ...mappedPropsToVueProps(mappedProps),
    ...props,
  },
}
</script>
