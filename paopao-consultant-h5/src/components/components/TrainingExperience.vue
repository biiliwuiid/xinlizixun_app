<template>
  <div id="training-experience">
    <!-- 培训经历开始 -->
    <div v-for="(item,index) in trainingExperienceList" :key="index" class="center">
      <div class="center-top">
        <img src="../../assets/images/zou.png">
        培训经历
        <img src="../../assets/images/you.png">
      </div>
      <div class="app-flex">
        <div style="line-height: 1rem;">主办机构:</div>
        <van-field
          v-model="item.organization"
          class="nameInput"
          maxlength="20"
          center
          placeholder="请输入主办机构"
        />
      </div>
      <div class="app-flex">
        <div style="line-height: 1rem;">起止时间:</div>
        <div style="width: 4.82rem;" class="app-flex">
          <van-field
            v-model="item.startDate"
            class="nameInputs"
            center
            readonly
            placeholder="开始年月"
            @click="startTimePick(index)"
          />
          -
          <van-field
            v-model="item.endDate"
            class="nameInputs"
            center
            readonly
            placeholder="结束年月"
            @click="endTimePick(index)"
          />
        </div>
        <van-popup v-model="startTimePicker" position="bottom">
          <van-datetime-picker
            v-model="currentDate"
            type="year-month"
            :min-date="minDate"
            :max-date="maxDate"
            @cancel="startTimePicker = false"
            @confirm="onStartTimeConfirm"
          />
        </van-popup>
        <van-popup v-model="endTimePicker" position="bottom">
          <van-datetime-picker
            v-model="currentDate"
            type="year-month"
            :min-date="minDate"
            :max-date="maxDate"
            @cancel="endTimePicker = false"
            @confirm="onEndTimeConfirm"
          />
        </van-popup>
      </div>
      <div class="app-flex">
        <div style="line-height: 1rem;">课程名称:</div>
        <van-field
          v-model="item.courseName"
          class="nameInput"
          maxlength="30"
          center
          placeholder="请输入课程名称"
        />
      </div>
      <div style="display: flex;justify-content: space-between;">
        <div style="line-height: 1rem;">培训证书:</div>
        <div style="width: 4.82rem;">
          <div v-if="!item.certificateUrl" class="justImageFileInput" @click="uploadTrainingClick(index)">
            <van-button class="justInput" />
          </div>
          <div v-else @click="uploadTrainingClick(index)">
            <img class="upload-image" :src="baseUrl + item.certificateUrl">
          </div>
          <p style="line-height: .5rem;font-size: .2rem;color: #666;">机构开具的任何证明资料（JPG/PNG）</p>
        </div>
      </div>
      <input id="training" multiple="false" type="file" accept="image/png,image/jpeg,image/jpg" style="display: none" @change="trainingImageChange">
      <div class="app-flex1">
        <div style="line-height: .6rem;">证明人:</div>
        <div style="padding-bottom: .2rem;">
          <van-field
            v-model="item.certifier"
            class="nameInput"
            maxlength="30"
            center
            placeholder="姓名+电话"
          />
          <p style="line-height: .5rem;font-size: .2rem;color: #666;">若无证件，请填写证明人姓名+电话</p>
        </div>
      </div>
      <div class="app-flex">
        <van-button class="Preservation" :loading="loading" loading-text="保存中..." @click="addTrainingExperience(index)">保存</van-button>
        <van-button v-if="index != 0" :loading="loading" loading-text="删除中..." class="delete" @click="minusTrainingRow(index)">删除此项</van-button>
      </div>
    </div>
    <van-button v-if="trainingExperienceList.length < 3" class="increase" @click="plusTrainingRow">增加一项</van-button>
    <!-- 培训经历结束 -->
  </div>
</template>
<script>
import Axios from 'axios'
import { fileUploadUrl, baseUrl,loadUrl } from '@/data/config'
import { listTrainingExperience, addTrainingExperience, deleteTrainingExperience } from '@/api/training'
export default {
  data() {
    return {
      baseUrl: baseUrl,
      consultantId: localStorage.getItem('CONSULTANT_ID'),
      loading: false,
      trainingExperienceList: [],
      trainingIndex: 0,
      startTimePicker: false,
      endTimePicker: false,
      minDate: new Date(2000, 0, 1),
      maxDate: new Date(),
      currentDate: new Date()
    }
  },
  watch: {
    'trainingExperienceList': {
      handler(newVal) {
        this.$emit('getTrainingExperienceData', newVal)
      },
      deep: true,
      immediate: true
    }
  },
  created() {
    this.initTrainingData()
  },
  methods: {
    initTrainingData() {
      listTrainingExperience(this.consultantId).then(r => {
        if (r.result.length > 0) {
          this.trainingExperienceList = r.result
        } else {
          var trainingExperience = { id: 0, consultantId: this.consultantId }
          this.trainingExperienceList.push(trainingExperience)
        }
      })
    },
    plusTrainingRow() {
      var tempObject = { id: 0, consultantId: this.consultantId }
      this.trainingExperienceList.push(tempObject)
    },
    minusTrainingRow(index) {
      if (this.trainingExperienceList[index].id > 0) {
        this.loading = true
        deleteTrainingExperience(this.trainingExperienceList[index].id).then(r => {
          this.loading = false
        })
      }
      this.trainingExperienceList.splice(index, 1)
    },
    addTrainingExperience(index) {
      this.loading = true
      this.trainingExperienceList[index].consultantId = this.consultantId
      addTrainingExperience(this.trainingExperienceList[index]).then(r => {
        if (r.success) {
          this.$toast.success('保存成功')
        } else {
          this.$notify(r.message)
        }
        this.loading = false
      })
    },
    startTimePick(index) {
      this.trainingIndex = index
      this.startTimePicker = true
    },
    endTimePick(index) {
      this.trainingIndex = index
      this.endTimePicker = true
    },
    onStartTimeConfirm(value) {
      this.trainingExperienceList[this.trainingIndex].startDate = this.timeFormat(value)
      this.startTimePicker = false
    },
    onEndTimeConfirm(value) {
      this.trainingExperienceList[this.trainingIndex].endDate = this.timeFormat(value)
      this.endTimePicker = false
    },
    timeFormat(time) { // 时间格式化 2019-09-08
      const year = time.getFullYear()
      const month = time.getMonth() + 1
      return year + '-' + (month < 10 ? '0' + month : month)
    },
    uploadTrainingClick(index) {
      this.trainingIndex = index
      document.getElementById('training').click()
    },
    trainingImageChange(e) {
      const file = e.target.files[0]
      if (file) {
        if (file.size > 0 && file.size <= 5485760) { // 判断文件大小
          var reader = new FileReader()
          reader.onload = function(event) {
            var txt = event.target.result
            var img = document.getElementById('training')
            img.src = txt
            img.style.backgroundRepeat = 'no-repeat'
          }
        } else {
          this.$notify('图片不能小于0kb且不能大于5mb')
          return
        }
      }
      reader.readAsDataURL(file)
      const param = new FormData() // 创建form对象
      param.append('file', file) // 通过append向form对象添加数据
      param.append('output', 'json') // 添加form表单中其他数据
      param.append('path', '/image') // 添加form表单中其他数据
      param.append('scene', 'image') // 添加form表单中其他数据
      const config = {
        headers: { 'Content-Type': 'multipart/form-data' }
      } 
      this.$set(this.trainingExperienceList[this.trainingIndex], 'certificateUrl', loadUrl)
      // 添加请求头
      Axios.post(fileUploadUrl, param, config).then(
        res => {
          this.$set(this.trainingExperienceList[this.trainingIndex], 'certificateUrl', res.data.path)
        }
      )
    }
  }
}
</script>
<style scoped>
@import "../../assets/public.css";
.center {
  border: 0.02rem solid rgba(157, 220, 177);
  border-radius: 0.2rem;
  width: 6.2rem;
  padding: 0.38rem 0.38rem;
  margin: 0.4rem auto;
  font-size: 0.24rem;
  color: #000;
}

.centers {
  display: flex;
  justify-content: center;
  border: 0.02rem solid rgba(157, 220, 177);
  border-radius: 0.2rem;
  width: 6.9rem;
  padding: 0.38rem 0;
  margin: 0.4rem auto;
  font-size: 0.24rem;
  color: #000;
}

.center-top {
  font-size: 0.26rem;
  display: flex;
  justify-content: center;
  padding-bottom: 0.2rem;
  color: #9edcb0;
}

.center-top img {
  width: 0.34rem;
  height: 0.26rem;
  margin: 0 0.2rem;
}

.center-tops {
  font-size: 0.2rem;
  text-align: center;
  padding-bottom: 0.2rem;
  color: #666;
}

#main {
  width: 4.82rem;
  height: 0.72rem;
  border: 0.02rem solid #f0f0f0;
  border-radius: 0.1rem;
}

#content {
  width: 3.82rem;
  height: 0.72rem;
  font-size: 0.24rem;
  background: #ffffff;
  line-height: 0.72rem;
  text-align: center;
  border-radius: 0.1rem;
}

#selectImg {
  top: 13px;
  right: 10px;
  cursor: pointer;
  width: 0.36rem;
  height: 0.12rem;
}

#selectItem {
  display: none;
  border: 0.01rem solid #eee;
  width: 4.82rem;
}

#selectItem ul {
  list-style: none;
  z-index: 10;
}

#selectItem ul li {
  height: 0.3rem;
  line-height: 0.3rem;
  padding-left: 0.1rem;
  background-color: #fff;
  font-size: 0.2rem;
  z-index: 10;
}

#selectItem ul li:hover {
  background-color: #ebebeb;
}

.cj_btn {
  width: 4.82rem;
  font-size: 0.24rpx;
  display: flex;
  align-items: center;
  justify-content: flex-start;
}

input[type="radio"] + label::before {
  content: " ";
  /*不换行空格*/
  display: inline-block;
  vertical-align: middle;
  font-size: 18px;
  width: 0.3rem;
  height: 0.3rem;
  margin-right: 0.1rem;
  border-radius: 50%;
  background: url(../../assets/images/yes.png);
  background-size: 0.3rem 0.3rem;
  /*原始按钮图片地址*/
}

input[type="radio"]:checked + label::before {
  content: " ";
  /*不换行空格*/
  display: inline-block;
  vertical-align: middle;
  font-size: 18px;
  width: 0.3rem;
  height: 0.3rem;
  margin-right: 0.1rem;
  border-radius: 50%;
  background: url(../../assets/images/no.png);
  background-size: 0.3rem 0.3rem;
  /*选中的按钮图片地址*/
}

input[type="radio"] {
  position: absolute;
  clip: rect(0, 0, 0, 0);
}

.onlable {
  margin-right: 0.4rem;
}
.cj_btn select {
  width: 4.82rem;
  height: 0.72rem;
  appearance: none;
  -moz-appearance: none;
  -webkit-appearance: none;
  background: url(../../assets/images/t6.png) no-repeat right;
  background-size: 0.36rem 0.12rem;
  font-size: 0.2rem;
  font-family: Microsoft YaHei;
  color: #666;
  outline: none;
  border: 0.02rem solid #f0f0f0;
  border-radius: 0.1rem;
  padding-left: 0.2rem;
}

.cj_btns {
  width: 4.82rem;
  display: flex;
  align-items: center;
  justify-content: flex-start;
}

.cj_btns select {
  width: 1.58rem;
  height: 0.72rem;
  appearance: none;
  -moz-appearance: none;
  -webkit-appearance: none;
  background: url(../../assets/images/t6.png) no-repeat right;
  background-size: 0.36rem 0.12rem;
  font-size: 0.2rem;
  font-family: Microsoft YaHei;
  color: #666;
  outline: none;
  border: 0.02rem solid #f0f0f0;
  border-radius: 0.1rem;
  padding-left: 0.2rem;
}

.nameInput {
  width: 4.64rem;
  height: 0.72rem;
  border: 0.02rem solid #f0f0f0;
  border-radius: 0.1rem;
  font-size: 0.24rem;
  background: #ffffff;
  padding-left: 0.18rem;
}

.nameInputs {
  width: 2.1rem;
  height: 0.72rem;
  border: 0.02rem solid #f0f0f0;
  border-radius: 0.1rem;
  font-size: 0.24rem;
  background: #ffffff;
  padding-left: 0.18rem;
}

/*定义图像以及大小*/
.justImageFileInput {
  width: 3rem;
  height: 1.5rem;
  border-radius: 0.1rem 0.1rem 0 0;
  background-image: url(../../assets/images/jia.png);
  background-size: 100% 1.5rem;
  /*这里可以换成图片路径（background-image：../img/....）注意图片路径*/
}
.upload-image {
  width: 3rem;
  height: 1.5rem;
  border-radius: 0.1rem 0.1rem 0 0;
  background-size: 100% 1.5rem;
  /*这里可以换成图片路径（background-image：../img/....）注意图片路径*/
}

.justInput {
  width: 3rem;
  height: 1.5rem;
  opacity: 0;
}

.Preservation {
  width: 3.78rem;
  height: 0.76rem;
  line-height: 0.76rem;
  color: #ffffff;
  background: #9ddcb0;
  text-align: center;
  border-radius: 0.2rem;
  font-size: 0.28rem;
  border: 0.02rem solid #9ddcb0;
  flex: 1;
}

.Preservations {
  width: 6.2rem;
  height: 0.76rem;
  line-height: 0.76rem;
  color: #ffffff;
  background: #9ddcb0;
  text-align: center;
  border-radius: 0.2rem;
  font-size: 0.28rem;
  margin: 0 auto;
}

.delete {
  width: 2.14rem;
  height: 0.76rem;
  line-height: 0.76rem;
  color: #9ddcb0;
  border: 0.02rem solid #9ddcb0;
  border-radius: 0.2rem;
  font-size: 0.28rem;
  text-align: center;
}

.increase {
  width: 6.9rem;
  height: 0.76rem;
  margin: 0.4rem auto;
  line-height: 0.76rem;
  color: #9ddcb0;
  border: 0.02rem solid #9ddcb0;
  border-radius: 0.38rem;
  font-size: 0.28rem;
  text-align: center;
  display: block;
  background: #fff;
}

</style>
