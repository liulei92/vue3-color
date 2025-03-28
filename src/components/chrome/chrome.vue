<script>
import colorMixin from '@/mixin/color';
import editableInput from '@/components/editable-input';
import saturation from '@/components/saturation';
import hue from '@/components/hue';
import alpha from '@/components/alpha';
import checkboard from '@/components/checkboard';

export default {
  name: 'Chrome',
  components: {
    Saturation: saturation,
    Hue: hue,
    Alpha: alpha,
    EdIn: editableInput,
    Checkboard: checkboard,
  },
  mixins: [colorMixin],
  props: {
    disableAlpha: {
      type: Boolean,
      default: false,
    },
    disableFields: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      fieldsIndex: 0,
      highlight: false,
    };
  },
  computed: {
    hsl() {
      const { h, s, l } = this.colors.hsl;
      return {
        h: h.toFixed(),
        s: `${(s * 100).toFixed()}%`,
        l: `${(l * 100).toFixed()}%`,
      };
    },
    activeColor() {
      const { rgba } = this.colors;
      return `rgba(${[rgba.r, rgba.g, rgba.b, rgba.a].join(',')})`;
    },
    hasAlpha() {
      return this.colors.a < 1;
    },
  },
  methods: {
    childChange(data) {
      this.colorChange(data);
    },
    inputChange(data) {
      if (!data)
        return;

      if (data.hex) {
        this.isValidHex(data.hex) && this.colorChange({
          hex: data.hex,
          source: 'hex',
        });
      }
      else if (data.r || data.g || data.b || data.a) {
        this.colorChange({
          r: data.r || this.colors.rgba.r,
          g: data.g || this.colors.rgba.g,
          b: data.b || this.colors.rgba.b,
          a: data.a || this.colors.rgba.a,
          source: 'rgba',
        });
      }
      else if (data.h || data.s || data.l) {
        const h = (data.h && /^\d+$/.test(data.h)) ? parseInt(data.h) : this.colors.hsl.h;
        const s = data.s ? (data.s.replace('%', '') / 100) : this.colors.hsl.s;
        const l = data.l ? (data.l.replace('%', '') / 100) : this.colors.hsl.l;

        this.colorChange({
          h,
          s,
          l,
          source: 'hsl',
        });
      }
    },
    toggleViews() {
      if (this.fieldsIndex >= 2) {
        this.fieldsIndex = 0;
        return;
      }
      this.fieldsIndex++;
    },
    showHighlight() {
      this.highlight = true;
    },
    hideHighlight() {
      this.highlight = false;
    },
  },
};
</script>

<template>
  <div role="application" aria-label="Chrome color picker" class="vc-chrome" :class="[disableAlpha ? 'vc-chrome__disable-alpha' : '']">
    <div class="vc-chrome-saturation-wrap">
      <Saturation :value="colors" @change="childChange" />
    </div>
    <div class="vc-chrome-body">
      <div class="vc-chrome-controls">
        <div class="vc-chrome-color-wrap">
          <div :aria-label="`current color is ${colors.hex}`" class="vc-chrome-active-color" :style="{ background: activeColor }" />
          <Checkboard v-if="!disableAlpha" />
        </div>

        <div class="vc-chrome-sliders">
          <div class="vc-chrome-hue-wrap">
            <Hue :value="colors" @change="childChange" />
          </div>
          <div v-if="!disableAlpha" class="vc-chrome-alpha-wrap">
            <Alpha :value="colors" @change="childChange" />
          </div>
        </div>
      </div>

      <div v-if="!disableFields" class="vc-chrome-fields-wrap">
        <div v-show="fieldsIndex === 0" class="vc-chrome-fields">
          <!-- hex -->
          <div class="vc-chrome-field">
            <EdIn v-if="!hasAlpha" label="hex" :value="colors.hex" @change="inputChange" />
            <EdIn v-if="hasAlpha" label="hex" :value="colors.hex8" @change="inputChange" />
          </div>
        </div>
        <div v-show="fieldsIndex === 1" class="vc-chrome-fields">
          <!-- rgba -->
          <div class="vc-chrome-field">
            <EdIn label="r" :value="colors.rgba.r" @change="inputChange" />
          </div>
          <div class="vc-chrome-field">
            <EdIn label="g" :value="colors.rgba.g" @change="inputChange" />
          </div>
          <div class="vc-chrome-field">
            <EdIn label="b" :value="colors.rgba.b" @change="inputChange" />
          </div>
          <div v-if="!disableAlpha" class="vc-chrome-field">
            <EdIn label="a" :value="colors.a" :arrow-offset="0.01" :max="1" @change="inputChange" />
          </div>
        </div>
        <div v-show="fieldsIndex === 2" class="vc-chrome-fields">
          <!-- hsla -->
          <div class="vc-chrome-field">
            <EdIn label="h" :value="hsl.h" @change="inputChange" />
          </div>
          <div class="vc-chrome-field">
            <EdIn label="s" :value="hsl.s" @change="inputChange" />
          </div>
          <div class="vc-chrome-field">
            <EdIn label="l" :value="hsl.l" @change="inputChange" />
          </div>
          <div v-if="!disableAlpha" class="vc-chrome-field">
            <EdIn label="a" :value="colors.a" :arrow-offset="0.01" :max="1" @change="inputChange" />
          </div>
        </div>
        <!-- btn -->
        <div class="vc-chrome-toggle-btn" role="button" aria-label="Change another color definition" @click="toggleViews">
          <div class="vc-chrome-toggle-icon">
            <svg
              style="width:24px; height:24px"
              viewBox="0 0 24 24"
              @mouseover="showHighlight"
              @mouseenter="showHighlight"
              @mouseout="hideHighlight"
            >
              <path fill="#333" d="M12,18.17L8.83,15L7.42,16.41L12,21L16.59,16.41L15.17,15M12,5.83L15.17,9L16.58,7.59L12,3L7.41,7.59L8.83,9L12,5.83Z" />
            </svg>
          </div>
          <div v-show="highlight" class="vc-chrome-toggle-icon-highlight" />
        </div>
        <!-- btn -->
      </div>
    </div>
  </div>
</template>

<style>
.vc-chrome {
  background: #fff;
  border-radius: 2px;
  box-shadow: 0 0 2px rgba(0,0,0,.3), 0 4px 8px rgba(0,0,0,.3);
  box-sizing: initial;
  width: 225px;
  font-family: Menlo;
  background-color: #fff;
}
.vc-chrome-controls {
  display: flex;
}
.vc-chrome-color-wrap {
  position: relative;
  width: 36px;
}
.vc-chrome-active-color {
  position: relative;
  width: 30px;
  height: 30px;
  border-radius: 15px;
  overflow: hidden;
  z-index: 1;
}
.vc-chrome-color-wrap .vc-checkerboard {
  width: 30px;
  height: 30px;
  border-radius: 15px;
  background-size: auto;
}
.vc-chrome-sliders {
  flex: 1;
}
.vc-chrome-fields-wrap {
  display: flex;
  padding-top: 16px;
}
.vc-chrome-fields {
  display: flex;
  margin-left: -6px;
  flex: 1;
}
.vc-chrome-field {
  padding-left: 6px;
  width: 100%;
}
.vc-chrome-toggle-btn {
  width: 32px;
  text-align: right;
  position: relative;
}
.vc-chrome-toggle-icon {
  margin-right: -4px;
  margin-top: 12px;
  cursor: pointer;
  position: relative;
  z-index: 2;
}
.vc-chrome-toggle-icon-highlight {
  position: absolute;
  width: 24px;
  height: 28px;
  background: #eee;
  border-radius: 4px;
  top: 10px;
  left: 12px;
}
.vc-chrome-hue-wrap {
  position: relative;
  height: 10px;
  margin-bottom: 8px;
}
.vc-chrome-alpha-wrap {
  position: relative;
  height: 10px;
}
.vc-chrome-hue-wrap .vc-hue {
  border-radius: 2px;
}
.vc-chrome-alpha-wrap .vc-alpha-gradient {
  border-radius: 2px;
}
.vc-chrome-hue-wrap .vc-hue-picker, .vc-chrome-alpha-wrap .vc-alpha-picker {
  width: 12px;
  height: 12px;
  border-radius: 6px;
  transform: translate(-6px, -2px);
  background-color: rgb(248, 248, 248);
  box-shadow: 0 1px 4px 0 rgba(0, 0, 0, 0.37);
}
.vc-chrome-body {
  padding: 16px 16px 12px;
  background-color: #fff;
}
.vc-chrome-saturation-wrap {
  width: 100%;
  padding-bottom: 55%;
  position: relative;
  border-radius: 2px 2px 0 0;
  overflow: hidden;
}
.vc-chrome-saturation-wrap .vc-saturation-circle {
  width: 12px;
  height: 12px;
}

.vc-chrome-fields .vc-input__input {
  font-size: 11px;
  color: #333;
  width: 100%;
  border-radius: 2px;
  border: none;
  box-shadow: inset 0 0 0 1px #dadada;
  height: 21px;
  text-align: center;
}
.vc-chrome-fields .vc-input__label {
  text-transform: uppercase;
  font-size: 11px;
  line-height: 11px;
  color: #969696;
  text-align: center;
  display: block;
  margin-top: 12px;
}

.vc-chrome__disable-alpha .vc-chrome-active-color {
  width: 18px;
  height: 18px;
}
.vc-chrome__disable-alpha .vc-chrome-color-wrap {
  width: 30px;
}
.vc-chrome__disable-alpha .vc-chrome-hue-wrap {
  margin-top: 4px;
  margin-bottom: 4px;
}
</style>
