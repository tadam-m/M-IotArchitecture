<template>
      <div id="popup-ampoule">
        <div id="light-icon-close" v-on:click="closePopup">
          <svg class="svg-icon" viewBox="0 0 20 20">
            <path d="M14.776,10c0,0.239-0.195,0.434-0.435,0.434H5.658c-0.239,0-0.434-0.195-0.434-0.434s0.195-0.434,0.434-0.434h8.684C14.581,9.566,14.776,9.762,14.776,10 M18.25,10c0,4.558-3.693,8.25-8.25,8.25c-4.557,0-8.25-3.691-8.25-8.25c0-4.557,3.693-8.25,8.25-8.25C14.557,1.75,18.25,5.443,18.25,10 M17.382,10c0-4.071-3.312-7.381-7.382-7.381C5.929,2.619,2.619,5.93,2.619,10c0,4.07,3.311,7.382,7.381,7.382C14.07,17.383,17.382,14.07,17.382,10"></path>
          </svg>
        </div>
        <div class="popup-header">
            <img src="../assets/icon/tips.png">
        </div>
        <div class="popup-body">
          <div id="body-color-on-off">
            <p class="title-str">POWER :</p>
            <Toggle :_val=_currentLightToggle :_id=_currentNameObject  v-on:toggleObject=toggleObject />
          </div>
          <div id="body-color-color">
            <p class="title-str">COLOR :</p>
            <input type="color" id="color-light" name="head" :value=_color>
          </div>
          <div id="body-intensity">
            <p class="title-str">INTENSITY :</p>
            <input type="number" id="color-intensity" name="head" :value=_intensity>
          </div>
          <div id="body-longitude">
            <p class="title-str">LONGITUDE :</p>
            <input type="text" id="longitude-input" name="head" :value=_longitude>
          </div>
          <div id="body-latitude">
            <p class="title-str">LATITUDE :</p>
            <input type="text" id="latitude-input" name="head" :value=_latitude>
          </div>
        </div>
        <div class="popup-footer">
          <button v-on:click="popupUpdateLight">UPDATE</button>
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
export default class PopupLight extends Vue {
    @Prop({default: true}) _currentLightToggle! : boolean;
    @Prop({default: ""}) _currentNameObject! : string;
    @Prop({default:""}) _color!: string;
    @Prop({default: 0}) _intensity!: number;
    @Prop({default: 0}) _latitude!: number;
    @Prop({default: 0}) _longitude!: number;

    currentLightToggle :boolean = false;
    currentNameObject : string = "";
    color: string ="";
    intensity: number = 0;
    longitude: number = 0;
    latitdue: number = 0;

    popupUpdateLight() {
        let colorIntensity = (<HTMLInputElement>document.getElementById("color-intensity"));
        let longitude = (<HTMLInputElement>document.getElementById("longitude-input"));
        let latitude = (<HTMLInputElement>document.getElementById("latitude-input"));

        if (colorIntensity) {
            this.longitude = parseFloat(longitude.value);
            this.latitdue = parseFloat(latitude.value);
            this.intensity = parseInt(colorIntensity.value);
            this.$emit("popupUpdateLight", this.currentLightToggle, this.intensity, this.longitude, this.latitdue);
        }
    }

    toggleObject(id:any, val: any) {
        this.currentLightToggle = val;
        this.$emit("toggleObject", this.currentNameObject, this.currentLightToggle);
    }

    closePopup() {
        this.$emit("closePopup");
    }

    mounted() {
        this.currentNameObject = this._currentNameObject;
        this.currentLightToggle = this._currentLightToggle;
        this.intensity = this._intensity;
        this.color = this._color;
        this.longitude = this._longitude;
        this.latitdue = this._latitude;
    }
}
</script>

<style lang="scss">
#popup-ampoule {
    position: absolute;
    font-family: 'Cyber';
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

    #light-icon-close {
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
      #body-color-on-off {
        display: flex;
        flex-direction: row;
        justify-content: left;
        align-items: center;
        color: rgb(13, 92, 196);
        .title-str {
          margin-right: 20px;
        }
      }
      #body-color-color {
        display: flex;
        flex-direction: row;
        justify-content: left;
        align-items: center;
        color: rgb(13, 196, 150);
        .title-str {
          margin-right: 20px;
        }
        input {
          border: none;
        }
      }
      #body-intensity {
          display: flex;
          flex-direction: row;
          justify-content: left;
          align-items: center;
          .title-str {
            margin-right: 20px;
            color: rgb(111, 0, 255);
            }
          input {
              max-width: 40px;
              font-size: 18px;
          }
      }
      #body-latitude, #body-longitude {
          display: flex;
          flex-direction: row;
          justify-content: left;
          align-items: center;
          .title-str {
            margin-right: 20px;
            color: gold;
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