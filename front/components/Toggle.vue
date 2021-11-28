<template>
    <input type="checkbox" class="toggle" v-on:click="toggle" id="tog1">
</template>

<style lang="sass">
input[type="checkbox"]
  -webkit-appearance: none
  -moz-appearance: none
  appearance: none
  -webkit-tap-highlight-color: transparent
  cursor: pointer
  &:focus
    outline: 0

.toggle
  height: 32px
  width: 52px
  border-radius: 16px
  display: inline-block
  position: relative
  margin: 0
  border: 2px solid #474755
  background: linear-gradient(180deg, #2D2F39 0%, #1F2027 100%)
  transition: all .2s ease
  &:after
    content: ''
    position: absolute
    top: 2px
    left: 2px
    width: 24px
    height: 24px
    border-radius: 50%
    background: white
    box-shadow: 0 1px 2px rgba(44,44,44,.2)
    transition: all .2s cubic-bezier(.5,.1,.75,1.35)
  &:checked
    border-color: #654FEC
    &:after
      transform: translatex(20px)
</style>

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
export default class Toggle extends Vue {
    @Prop({default: false}) _val!: Boolean;
    @Prop({default: ""}) _id!: String;
    _valueToggle : any = false;

    toggle() {
        this._valueToggle = !this._valueToggle;
        this.$emit("toggleObject", this._id, this._valueToggle);
    }

    mounted() {
        this._valueToggle = this._val;
        let obj = (<HTMLInputElement>document.getElementById("tog1"));
        
        if (obj) obj.checked = this._valueToggle;
    }
}
</script>