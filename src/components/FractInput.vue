<template>

  <div class="frac-input">
    <input type="text" v-model="numerator" @keypress="mask" :readonly="readonly"/>
    <input type="text" v-model="denominator" @keypress="mask" :readonly="readonly" />
    <div v-if="removable" @click="$emit('remove', id)" class="frac-input-remove">&times;</div>
  </div>

</template>

<script>
  import { regexp, regexpresult } from './regexp.js';

  export default {
    name: 'FracInput',

    props: {
      id: {
        type: String,
      },
      data: {
        type: String,
      },
      removable: {
        type: Boolean,
        default: false,
      },
      readonly: {
        type: Boolean,
        default: false,
      },
    },

    created() {
      this.setData(); 
    },

    data() {
      return {
        numerator: '',
        denominator: '',
      }
    },

    methods: {

      setData() {
        let regex = regexp;

        if ( this.readonly ) {
          regex = regexpresult;                        
        }

        if ( regex.test(this.data) ) {        
          let match = this.data.match(regex);
          this.numerator = match[1];
          this.denominator = match[2];
        }     
      },

      mask($event) {
        let keyCode = ($event.keyCode ? $event.keyCode : $event.which);

        // 0 ... 9
        if (keyCode < 48 || keyCode > 57) {
          $event.preventDefault();
        }     
      }
    },

    watch: {
      numerator: function(val, oldVal) {
        if (!this.readonly) {
          if (!val) {
            this.numerator = '';        
          } else {        
            if (val < 1 || val > 99) {
              this.numerator = oldVal;
            } else {
              this.numerator = val;
            }
          }      

          this.$emit('update', this.id, `${this.numerator}/${this.denominator}`);
        }
      },

      denominator: function(val, oldVal) {
        if (!this.readonly) {
          if (!val) {
            this.denominator = '';        
          } else {        
            if (val < 1 || val > 99) {
              this.denominator = oldVal;
            } else {
              this.denominator = val;
            }
          }

          this.$emit('update', this.id, `${this.numerator}/${this.denominator}`);
        }
      },

      data: function() {
        this.setData();
      }
    }
  }
</script>

<style lang="scss">

  @import "src/assets/styles/_variables.scss";

  .frac {

    &-input {
      position: relative;
      margin: 0 8px;
      padding: 8px 16px;
      
      &:after {
        content: '';
        display: block;
        width: 100%;
        height: 2px;
        background: $font-color;
        position: absolute;
        top: 50%;
        left: 0;
        margin: -1px 0 0 0;
      }

      input {
        display: block;
        border: 2px solid $font-color;
        border-top: none;
        border-radius: 0 0 $border-radius $border-radius;
        color: $font-color;
        background: $brand-color;
        font-size: 24px;
        padding: 8px;
        width: 40px;
        text-align: center;

        &:first-child {
          border: 2px solid $font-color;
          border-bottom: none;
          border-radius: $border-radius $border-radius 0 0;
        }
      }

      &.result {
        display: inline-block;
        vertical-align: middle;

        input {
          border-color: transparent;
        }
      }

      &-remove {
        position: absolute;
        font-size: 36px;
        top: -32px;
        right: -12px;
        cursor: pointer;
        opacity: .7;
        transition: opacity .15s ease-in-out;

        &:hover {
          opacity: 1;
        }
      }
    }
  }

</style>
