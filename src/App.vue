<template>
  <div id="app">
    <div class="frac-form">

      <div class="frac-formula">

        <template v-for="(frac, index) in fractions">          
          <FractInput class="frac-formula-el" 
            :key="frac.id" 
            :id="frac.id" 
            :data="frac.value" 
            :removable="fractions.length > min"
            @remove="remove"
            @update="update"
          />

          <div class="frac-formula-el" :key="`sign-${frac.id}`">
            {{ (++index) == fractions.length ? ' = ' : ' + ' }}
          </div>
        </template>

        <div class="frac-formula-el result">
          {{ result.value }}
          <FractInput class="frac-formula-el result"
            v-if="!normalize && result.fraction"
            :data="result.fraction" 
            :readonly="true"
          />
        </div>    

      </div>

      <div class="frac-action">
        <button @click.prevent="add" :class="{ hidden: fractions.length >= max }">Add new element</button>
        <label for="normalize">
          <input type="checkbox" v-model="normalize" id="normalize" /> Normilize result ?
        </label>
      </div> 

    </div>    
  </div>
</template>

<script>
  import FractInput from './components/FractInput.vue';
  import { regexp } from './components/regexp.js';

  let fractionsExample = [
    {
      'id': 'id-1',
      'value': '99/5'
    },
    {
      'id': 'id-2',
      'value': '71/4'
    },
  ];

  export default {
    name: 'App',

    components: {
      FractInput
    },

    created() {
      this.fractions = fractionsExample;
    },

    data() {
      return {
        fractions: [],
        min: 2,
        max: 5,
        normalize: false,
      }
    },

    computed: {
      result() {
        
        let res = {
          'value': 0,
          'fraction': '',
        };
        
        for (let frac of this.fractions) {
          if ( regexp.test(frac.value) ) {
            let match = frac.value.match(regexp);
            res.value += match[1]/match[2];          
          }
        }

        if (res.value) {
          if (this.normalize) {
            res.value = res.value.toFixed(2)          
          } else {
            let fraction = (res.value - Math.trunc(res.value)).toFixed(2);

            var len = fraction.toString().length - 2;

            let den = Math.pow(10, len);
            let num = fraction * den;

            var divisor = this.gcd(num, den); 

            num /= divisor;
            den /= divisor;

            if (num && den) {
              res.fraction = `${Math.floor(num)}/${Math.floor(den)}`;
            }           
            
            res.value = (Math.trunc(res.value) > 0 ) ? parseInt(res.value) : ''; 
          }
        }

        return res ?? ''; 
      },    
    },

    methods: {

      gcd(a, b) {
        if (b < 0.0000001) return a;
        return this.gcd(b, Math.floor(a % b)); 
      },

      add() {
        if (this.fractions.length < this.max) {
          this.fractions.push({'id': 'id-' + new Date().getTime()});
        }
      },

      remove(id) {
        if (this.fractions.length > this.min) {
          let filtered = this.fractions.filter((frac) => frac.id !== id)
          this.fractions = filtered;
        }
      },

      update(id, data) {
        let index = this.fractions.map(filtered => filtered.id).indexOf(id);
        let frac = {};
        frac.id = id;
        frac.value = data;
        this.fractions.splice(index, 1, frac);
      },
    },

  }
</script>

<style lang="scss">

  @import "src/assets/styles/_variables.scss";

  * {
    padding: 0;
    margin: 0;
    font-family: inherit;
  }

  html, body {
    height: 100%;
  }

  body {
    background-color: $brand-color;
    font-family: 'Roboto', sans-serif;
    color: $font-color;
    font-size: 1rem;
    font-weight: 400;
    line-height: 1.5;
  }

  #app {
    display: flex;
    flex-direction: row;
    flex-wrap: nowrap;
    justify-content: center;
    align-content: stretch;
    align-items: center;
    width: 100%;
    height: 100%;
    text-align: center;
  }

  .frac {

    &-formula {
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      justify-content: center;
      align-content: stretch;
      align-items: center;

      &-el {
        font-size: 46px;
        padding: 8px;

        &.result {
          font-size: 32px;
        }
      }
    }

    &-action {

      .hidden {
        visibility: hidden;
      }

      button {
        padding: 8px 16px;
        margin: 16px auto;
        height: 48px;
        line-height: 32px;
        font-size: 22px;
        color: $font-color;
        background: $brand-color;
        border: 2px solid $font-color;
        border-radius: $border-radius;
        cursor: pointer;
        transition: color .15s ease-in-out, 
                    background-color .15s ease-in-out;
        &:hover {
          color: $brand-color;
          background: $font-color;
        }
      }

      label {
        margin: 0 16px;
      }
    }

  }

</style>
