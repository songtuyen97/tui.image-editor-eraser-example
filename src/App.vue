<template>
  <div id="app">
    <div style="height: 80vh;">
      <tui-image-editor ref="tuiImageEditor" :include-ui="useDefaultUI" :options="options"></tui-image-editor>
      <div style="margin-top: 20px; display: flex;">
        <button @click="isEraser = !isEraser" :style="{background: isEraser ? 'green' : ''}">Eraser</button>
      </div>

    </div>
  </div>
</template>

<script>
import 'tui-color-picker/dist/tui-color-picker.css';
import 'tui-image-editor/dist/tui-image-editor.css';
import { ImageEditor } from '@toast-ui/vue-image-editor';
import { fabric } from 'fabric';

export default {
  name: 'App',
  components: {
    'tui-image-editor': ImageEditor,
  },
  data() {
    return {
      useDefaultUI: true,
      options: {
        // for tui-image-editor component's "options" prop
        cssMaxWidth: 700,
        cssMaxHeight: 500,
      },
      isEraser: false,
      isMouseDown: false,
      imageBase64: '',
    };
  },
  computed: {
    editorInstance() {
      return this.$refs.tuiImageEditor.editorInstance;
    },
    canvas() {
      return this.editorInstance._graphics.getCanvas();
    }
  },
  methods: {
    eraserListener() {
      const self = this;
      self.canvas.freeDrawingBrush2 = {
        width: 20,
        onMouseDown: function (pointer) {
          if (self.isEraser && self.isMouseDown) {
            const circle = new fabric.Circle({
              left: pointer.x,
              top: pointer.y,
              radius: this.width / 2,
              selectable: false,
              globalCompositeOperation: 'destination-out'
            });
            self.canvas.add(circle);
          }
        },

        onMouseMove: function (pointer) {
          if (self.isEraser && self.isMouseDown) {
            const circle = new fabric.Circle({
              left: pointer.x,
              top: pointer.y,
              radius: this.width / 2,
              selectable: false,
              globalCompositeOperation: 'destination-out'
            });
            self.canvas.add(circle);
          }
        },

        onMouseUp: function () {
          self.canvas.isDrawingMode2 = false;

          fabric.Image.fromURL(self.imageBase64, (img) => {
            img.set({
              scaleX: self.canvas.width / img.width,
              scaleY: self.canvas.height / img.height,
              globalCompositeOperation: 'destination-over',
              crossOrigin: 'anonymous'
            });
            self.canvas.add(img);
          }, { crossOrigin: 'anonymous' });
        }
      };

      this.canvas.on('mouse:down', (event) => this.canvas.freeDrawingBrush2.onMouseDown(self.canvas.getPointer(event.e)));
      this.canvas.on('mouse:move', (event) => this.canvas.freeDrawingBrush2.onMouseMove(self.canvas.getPointer(event.e)));
      this.canvas.on('mouse:up', () => this.canvas.freeDrawingBrush2.onMouseUp());
    },
    upImage() {
      this.$refs.tuiImageEditor.invoke('getDrawingMode');
      this.$refs.tuiImageEditor.invoke('loadImageFromURL', 'https://cellphones.com.vn/sforum/wp-content/uploads/2024/02/anh-thien-nhien-90.jpg', 'My sample image');
    },
    toBase64 (file) {
      return new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.readAsDataURL(file);
          reader.onload = () => resolve(reader.result);
          reader.onerror = reject;
      })
    }
  },
  mounted() {
    this.eraserListener();
    document.addEventListener('mousedown', () => this.isMouseDown = true);
    document.addEventListener('mouseup', () => this.isMouseDown = false);
    document.querySelector('.tui-image-editor-header-buttons .tui-image-editor-load-btn')?.addEventListener('change',async  (e) => {
      this.imageBase64 = await this.toBase64(e.target.files[0]);
    })
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
