<template>
  <div id="app">
    <Row class="grid-tool__toolbar">
      <Column medium="1:3">
        <div class="input-field">
          <label>
            Image url
          </label>
          <input
            type="text"
            placholder="Give the url to an image"
            v-model="url"
            @change="checkUrl"
          />
        </div>
      </Column>
      <Column medium="2:3">
        <div class="input-group">
          <div class="input-field">
            <label>
              Grid width
            </label>
            <input
              type="number"
              v-model="grid.width"
              step="10"
              @change="updateSettings"
            />
          </div>
          <div class="input-field">
            <label>
              Total columns
            </label>
            <input
              type="number"
              v-model="grid.columns"
              @change="updateSettings"
            />
          </div>
          <div class="input-field">
            <label>
              Grid Offset
            </label>
            <input
              type="number"
              v-model="grid.offset_left"
              step="10"
              @change="updateSettings"
            />
          </div>
          <div class="input-field">
            <label>
              Zoom
            </label>
            <select
              v-model="grid.zoom"
              @change="updateSettings"
            >
              <option value="0.1">10%</option>
              <option value="0.25">25%</option>
              <option value="0.5">50%</option>
              <option value="0.75">75%</option>
              <option
                value="1"
                selected
              >100%</option>
              <option value="1.5">150%</option>
              <option value="2">200%</option>
            </select>
          </div>
          <div class="input-field">
            <label>
              Grid type
            </label>
            <select
              v-model="grid.type"
              @change="updateSettings"
            >
              <option value="lines">Lines</option>
              <option value="odd-even">Odd/Even</option>
            </select>
          </div>
        </div>
      </Column>
    </Row>
    <div class="grid-tool__view">
      <div
        class="grid-tool__image"
        v-if="theImage"
      >
        <img
          :src="theImage"
          alt="the image preview"
        />
        <div
          class="grid-tool__grid"
          :class="`grid-tool__grid--${grid.type}`"
        ></div>
      </div>
      <div
        class="grid-tool__start"
        v-if="!theImage"
      >

        <div class="grid-tool__dropzone">
          <div class="grid-tool__dropzone-title">Drop your image here</div>
          <input
            type="file"
            @change="uploadImage"
          />
        </div>
      </div>
    </div>
    <section class="container">
      <div class="content">
        <h2>Grid Tool</h2>
        <h3>But, what does it do?</h3>
        <p>Sometimes, you see these designs and just want to know how the grid is build. </p><p>You can load your images in photoshop or sketch and figure it out, but sometimes a simple tool like this can help you out too :).</p>
       
      </div>
    </section>
    <silFooter></silFooter>
  </div>
</template>

<script>
/* eslint-disable */
// import axios from "axios";
import Grid from "@sil/grid";

import silFooter from "./components/footer.vue";
const Row = Grid.row;
const Column = Grid.column;

export default {
  name: "app",
  components: {
    Column,
    Row,
    silFooter
  },
  data() {
    return {
      url: null,
      showImage: false,
      theImage: null,
      blob: false,
      grid: {
        og_width: 0,
        width: 0,
        columns: 24,
        zoom: 1,
        offset_left: 0,
        type: "lines",
        color: `rgba(127, 127, 127, 1)`
      }
    };
  },
  created() {
    // this.updateSettings();
  },
  methods: {
    checkUrl: function() {
      if (this.url) {
        this.showimage = true;
        this.theImage = this.url;
        this.getImageSize();
        this.updateSettings();
        this.blob = false;
      } else {
        this.showImage = false;
      }
    },
    getImageSize() {
      const _this = this;
      const img = new Image();
      img.src = this.url;
      img.onload = function() {
        _this.grid.width = this.width;
        _this.grid.og_width = this.width;
      };
      if (_this.grid.width > window.innerWidth) {
        _this.grid.zoom = 0.5;
      }
    },
    uploadImage(e) {
      const _this = this;
      _this.url = URL.createObjectURL(e.srcElement.files[0]);
      _this.theImage = URL.createObjectURL(e.srcElement.files[0]);
      _this.blob = true;
      _this.getImageSize();
      setTimeout(function(){
        _this.updateSettings();
      },100);
    },
    updateSettings() {
      let view = document.querySelector(".grid-tool__view");
      view.style.setProperty(
        "--grid-width",
        this.grid.width * this.grid.zoom + "px"
      );
      view.style.setProperty("--grid-og-width", this.grid.og_width + "px");
      view.style.setProperty(
        "--grid-image-width",
        this.grid.og_width * this.grid.zoom + "px"
      );
      view.style.setProperty("--grid-columns", this.grid.columns);
      view.style.setProperty("--grid-color", this.grid.color);
      view.style.setProperty("--grid-zoom", this.grid.zoom);
      view.style.setProperty(
        "--grid-offset-left",
        this.grid.offset_left * this.grid.zoom + "px"
      );
    }
  }
};
</script>

<style lang="scss">
@import "~@sil/base-style/src/scss/index.full";

.grid-tool {
  &__toolbar {
    background-color: color(Black);
    padding: 1rem;
  }
  &__view {
    position: relative;
    left: 0;
    top: 0;
    display: block;
    padding: grid(1);
    @include min-(padding, 1, 40);
    min-height: 100vh;
    min-width: 100vw;
    background-color: color(Black, 0.85);
    overflow: scroll;
  }
  &__image {
    position: relative;
    display: block;
    margin: auto;
    width: var(--grid-image-width);
    overflow: hidden;
    img {
      display: block;
      width: var(--grid-image-width);
    }
  }
  &__grid {
    position: absolute;
    left: var(--grid-offset-left, 0);
    top: 0;
    display: block;
    width: var(--grid-width, 100%);
    height: 100%;
    box-shadow: 0 0 0 calc(var(--grid-image-width) - var(--grid-width)) black,
      0 0 0 10px black;
    &--lines {
      opacity: 0.5;
      background-image: linear-gradient(
        to right,
        var(--grid-color, rgb(127, 127, 127)),
        var(--grid-color, rgb(127, 127, 127)) calc(0% + 1px),
        transparent calc(0% + 1px),
        transparent
      );
      background-size: calc(100% / var(--grid-columns, 24));
    }
    &--odd-even {
      opacity: 0.25;
      background-image: linear-gradient(
        to right,
        var(--grid-color, rgb(127, 127, 127)),
        var(--grid-color, rgb(127, 127, 127)) calc(50%),
        transparent calc(50%),
        transparent
      );
      background-size: calc(100% / ((var(--grid-columns, 24) / 2)));
    }
  }
  &__start {
    width: 100%;
    height: 80vh;

    display: flex;
    align-items: center;
    justify-content: center;
  }
  &__dropzone {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    width: grid(5);
    height: grid(5);
    @include min-(width, 5, 320) {
      height: 320px;
    }
    background: color(White, 1);
    border-radius: 50%;
    overflow: hidden;
    input[type="file"] {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: color(Purple);
    }
  }
  &__dropzone-title {
    position: absolute;
    top: 50%;
    left: 50%;
    z-index: 10;
    transform: translate(-50%, -50%);
    color: color(White);
    font-size: grid(0.5);
    @include min-(null, 5, 320) {
      font-size: 32px;
    }
    font-weight: bold;
    pointer-events: none;
  }
}

.input-field {
  label {
    display: block;
    color: color(White);
    padding: 0.25rem 0.5rem;
    font-size: 14px;
  }
  select,
  input[type="text"],
  input[type="number"] {
    width: 100%;
    max-width: 100%;
    min-width: 100%;
    padding: 0.25rem 0.5rem;
    background-color: transparent;
    color: white;
    border: 1px solid color(White, 0.25);
  }
  & + .input-field {
    margin: 0;
    margin-left: 1rem;
  }
}
.input-group {
  display: flex;
}
</style>
