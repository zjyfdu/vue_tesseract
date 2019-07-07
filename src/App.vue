<template>
  <div id="app">
    <div>
      <img v-if="imageUrl" :src="imageUrl" id="text-img" />
    </div>
    <div>
      <el-upload
        class="img-uploader"
        action
        :show-file-list="false"
        :auto-upload="false"
        :on-change="imgPreview"
      >
        <el-button type="primary">上 传</el-button>
      </el-upload>
      <el-button v-if="imageUrl" class="img-btn" type="primary" @click="recognize">识 别</el-button>
      <el-button v-if="imageUrl" class="img-btn" type="info" @click="dialogVisible = true">裁 剪</el-button>
    </div>

    <div v-for="(value, key) in progress_list" class="pregress-status">
      <el-row justify="center">
        <el-col :span="4" :offset="6">
          <div class="grid-content">{{ key }}</div>
        </el-col>
        <el-col :span="8">
          <el-progress :text-inside="true" :stroke-width="20" :percentage="value" :key="key"></el-progress>
        </el-col>
      </el-row>
    </div>
    <div v-for="res in recog_result">
      <p>{{res}}</p>
    </div>

    <div>
      <el-dialog title="图片剪裁" :visible.sync="dialogVisible" append-to-body>
        <div class="cropper-content">
          <div class="cropper" style="text-align:center">
            <VueCropper
              ref="cropper"
              :img="imageUrl"
              :outputType="option.outputType"
              :info="true"
              :full="option.full"
              :canMove="option.canMove"
              :canMoveBox="option.canMoveBox"
              :original="option.original"
              :autoCrop="option.autoCrop"
              :fixedNumber="option.fixedNumber"
              :centerBox="option.centerBox"
              :infoTrue="option.infoTrue"
              :fixedBox="option.fixedBox"
            ></VueCropper>
          </div>
        </div>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="finish">确 认</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<style>
#app {
  text-align: center;
}
#text-img {
  max-width: 80%;
}
.img-uploader {
  display: inline;
}
.pregress-status {
  margin: 20px;
}
.img-btn {
  margin-left: 10px;
}
#text-img {
  display: inline-block;
}
.cropper {
  width: auto;
  height: 300px;
}
</style>

<script>
import Tesseract from "tesseract.js";
import { VueCropper } from "vue-cropper";

const { TesseractWorker } = Tesseract;
const worker = new TesseractWorker();

export default {
  data() {
    return {
      imageUrl: "",
      dialogVisible: false,
      // 裁剪组件的基础配置option
      option: {
        info: true, // 裁剪框的大小信息
        outputType: "jpeg", // 裁剪生成图片的格式
        canScale: false, // 图片是否允许滚轮缩放
        autoCrop: true, // 是否默认生成截图框
        fixedNumber: [7, 5], // 截图框的宽高比例
        full: true, // 是否输出原图比例的截图
        infoTrue: true, // true 为展示真实输出图片宽高 false 展示看到的截图框宽高
        centerBox: true
      },
      progress_list: {},
      recog_result: []
    };
  },
  components: {
    VueCropper
  },
  methods: {
    imgPreview(file) {
      this.imageUrl = URL.createObjectURL(file.raw);
      // this.dialogVisible = true;
    },
    recognize() {
      // this.progress_list = {};
      this.recog_result = [];
      const img = document.getElementById("text-img");
      console.log(img);
      worker
        .recognize(this.imageUrl, "eng+chi_sim")
        .progress(p => {
          console.log(p);
          if ("progress" in p) {
            // this.progress_list[p.status] = p.progress * 100; // Vue 不能监控这个
            this.$set(
              this.progress_list,
              p.status,
              Math.floor(p.progress * 100)
            );
          }
        })
        .then(result => {
          var t;
          clearTimeout(t);
          t = setTimeout(() => {
            this.progress_list = {};
          }, 1000);
          for (var par in result.lines) {
            this.recog_result.push(result.lines[par].text);
          }
          console.log(result);
        });
    },
    finish() {
      this.$refs.cropper.getCropBlob(data => {
        console.log(data);
        this.imageUrl = URL.createObjectURL(data);
        this.dialogVisible = false;
      });
    }
  }
};
</script>
