<template>
      <div id="popup-ink">
        <div id="ink-icon-close" v-on:click="closePopup">
          <svg class="svg-icon" viewBox="0 0 20 20">
            <path d="M14.776,10c0,0.239-0.195,0.434-0.435,0.434H5.658c-0.239,0-0.434-0.195-0.434-0.434s0.195-0.434,0.434-0.434h8.684C14.581,9.566,14.776,9.762,14.776,10 M18.25,10c0,4.558-3.693,8.25-8.25,8.25c-4.557,0-8.25-3.691-8.25-8.25c0-4.557,3.693-8.25,8.25-8.25C14.557,1.75,18.25,5.443,18.25,10 M17.382,10c0-4.071-3.312-7.381-7.382-7.381C5.929,2.619,2.619,5.93,2.619,10c0,4.07,3.311,7.382,7.381,7.382C14.07,17.383,17.382,14.07,17.382,10"></path>
          </svg>
        </div>
        <div class="popup-header">
            <img src="../assets/icon/humidity-sensor.png">
        </div>
        <div class="popup-body">
          <div id="body-ink">
            <p class="title-str">INK :</p>
            <input type="text" id="ink-input" name="head" :value=_ink>
          </div>
          <div id="body-substance">
            <p class="title-str">SUBSTANCE :</p>
            <input type="text" id="substance-input" name="head" :value=_substance>
          </div>
          <div id="body-fluid">
            <p class="title-str">FLUID :</p>
            <input type="number" id="fluid-input" name="head" :value=_fluid>
          </div>
          <div id="body-fluidmax">
            <p class="title-str">FLUIDMAX :</p>
            <input type="number" id="fluidmax-input" name="head" :value=_fluidMax>
          </div>
        </div>
        <div class="popup-footer">
          <button v-on:click="popupUpdateInk">UPDATE</button>
        </div>
      </div>
</template>

<script lang="ts">
import {
  Component,
  Inject,
  Model,
  Prop,
  Vue,
  Provide,
  Watch,
  Emit,
} from "nuxt-property-decorator"

@Component
export default class PopupInk extends Vue {
    @Prop({default: ""}) _ink! : string;
    @Prop({default: ""}) _substance! : string;
    @Prop({default:0}) _fluid!: number;
    @Prop({default: 0}) _fluidMax!: number;
    
    ink :string = "";
    substance : string = "";
    fluid: number =0;
    fluidMax: number = 0;

    popupUpdateInk() {
        let ink = (<HTMLInputElement>document.getElementById("ink-input"));
        let substance = (<HTMLInputElement>document.getElementById("substance-input"));
        let fluid = (<HTMLInputElement>document.getElementById("fluid-input"));
        let fluidMax = (<HTMLInputElement>document.getElementById("fluidmax-input"));
        
        if (ink && substance && fluid && fluidMax) {
            try {
                this.fluid = parseInt(fluid.value, 10);
                this.fluidMax = parseInt(fluidMax.value, 10);
                this.substance = substance.value;
                this.ink = ink.value;
                this.$emit("popupUpdateInk", this.ink, this.substance, this.fluid, this.fluidMax);
            } catch(e) {
                console.error(e);
            }
        }
    }

    toggleObject(id:any, val: any) {
       // this.$emit("toggleObject", this.currentNameObject, this.currentLightToggle);
    }

    closePopup() {
        this.$emit("closePopup");
    }

    mounted() {
        this.ink = this._ink;
        this._substance = this._substance;
        this.fluid = this._fluid;
        this.fluidMax = this._fluidMax;
    }
}
</script>

<style lang="scss">
#popup-ink {
    position: absolute;
    left:10%;
    padding: 20px;
    top:25%;
    background: white;
    border-radius: 5px;
    min-width: 200px;
    min-height: 300px;
    -webkit-animation: fadein 1s; /* Safari, Chrome and Opera > 12.1 */
       -moz-animation: fadein 1s; /* Firefox < 16 */
        -ms-animation: fadein 1s; /* Internet Explorer */
         -o-animation: fadein 1s; /* Opera < 12.1 */
            animation: fadein 1s;

    #ink-icon-close {
        position: absolute;
        right: 10px;
        top: 10px;
        &:hover {
          fill: hotpink;
        }
        .svg-icon {
          width: 35px;
          height: 35px;
          transition: 0.5s;
          fill: inherit;
          path {
            fill: inherit;
          }
        }
    }
    .popup-header {
      margin-top: 20px;
      position:relative;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      img {
        width: 80px;
      }
    }
    .popup-body {
      margin-top: 40px;
      #body-ink,#body-substance {
        display: flex;
        flex-direction: row;
        justify-content: left;
        align-items: center;
        color: rgb(13, 196, 150);
        .title-str {
          margin-right: 20px;
        }
        input {
            max-width: 100px;
        }
      }
      #body-fluid, #body-fluidmax {
          display: flex;
          flex-direction: row;
          justify-content: left;
          align-items: center;
          .title-str {
            margin-right: 20px;
            color: rgb(111, 0, 255);
            }
          input {
              max-width: 60px;
              font-size: 18px;
          }
      }
    }
    .popup-footer {
        margin-top: 20px;
        display: flex;
        justify-content: center;
        align-items: center;
    }
  }

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

@keyframes fadein {
    from { opacity: 0; }
    to   { opacity: 1; }
}

/* Firefox < 16 */
@-moz-keyframes fadein {
    from { opacity: 0; }
    to   { opacity: 1; }
}
</style>