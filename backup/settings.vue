<template>
  <div class="row justify-center">
    <q-dialog v-model="input_fail" persistent >
      <q-card>
        <q-card-section>
          <div class="text-h6"><b>提醒</b></div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          無法更新，請確認欄位是否正確填寫或上傳。
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="確認" color="primary" v-close-popup/>
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="update_fail" persistent >
      <q-card>
        <q-card-section>
          <div class="text-h6"><b>提醒</b></div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          無法更新，伺服器忙碌中，請重新再試。
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="確認" color="primary" v-close-popup @click="close_update_fail()"/>
        </q-card-actions>
      </q-card>
    </q-dialog>

    <div class="q-pa-md text-center" style="max-width: 400px">
      <q-form @submit="updateProfile()">
        <div class="row justify-center">
          <div class="col-12 q-pa-sm" style="height:92px">
            <q-input disable v-model="info.user_data.email" filled type="email" label="電子信箱" hint="你的電子信箱（登入帳號）">
              <template v-slot:prepend>
                <q-icon name="email" />
              </template>
            </q-input>
          </div>
          <!-- <p class="q-mb-sm"/> -->
          <div class="col-12 q-pa-sm">
            <q-input 
              v-model="info.user_data.password" 
              filled :type="isPwd ? 'password' : 'text'"  
              label="密碼"
              hint="你的用戶密碼"
              lazy-rules :rules="[ val => verify_columns('password', val)]"> 
              <template v-slot:prepend>
                <q-icon name="lock" />
              </template>
              <template v-slot:append>
                <q-icon
                :name="isPwd ? 'visibility_off' : 'visibility'"
                class="cursor-pointer"
                @click="isPwd = !isPwd">
                </q-icon>
              </template>
            </q-input>
          </div>
          <!-- <p class="q-mb-sm"/> -->
          <div class="col-12 q-pa-sm">
            <q-input 
              v-model="info.user_data.first_name" 
              filled type="text" 
              label="姓名"
              hint="你的用戶資訊"
              lazy-rules :rules="[ val => verify_columns('first_name', val)]">
              <template v-slot:prepend>
                <q-icon name="person" />
              </template>
            </q-input>
          </div>
          <!-- <p class="q-mb-sm"/> -->

          <div class="city-selector-set col col-12 q-pa-sm row justify-center">
              <!-- 縣市選單 -->
              <select v-model="county" class="county col-3" id="select_county" ></select>
              <!-- 區域選單 -->
              <select v-model="district" class="district col-3" id="select_district" ></select>
              <!-- 郵遞區號欄位 (建議加入 readonly 屬性，防止修改) -->
              <input v-model="info.user_data.postcode" class="zipcode col-3"  id="input_zipcode" type="text" size='8%' disabled placeholder="郵遞區號">
          </div>

          <div class="col col-12 q-pa-sm">
            <q-input              
              filled
              v-model="info.user_data.address_1"
              type="text"
              placeholder="地址"
              hint="你的聯絡地址"
              lazy-rules :rules="[ val => verify_columns('address_1', val)]">
              <template v-slot:prepend>
                <q-icon name = "location_on"></q-icon>
              </template>
            </q-input>
          </div>

          <div class="col-12 q-pa-sm">
            <q-input
              filled
              v-model="info.user_data.phone"
              label="聯絡電話"
              hint="你的手機號碼"
              mask="####-###-###"
              lazy-rules :rules="[ val => verify_columns('phone', val) ]">
              <template v-slot:prepend>
                <q-icon name="smartphone" />
              </template>
              <!-- <template v-slot:append>
                <q-icon name="close" @click="info.user_data.phone = ''" class="cursor-pointer" />
              </template> -->
            </q-input>
          </div>
          <!-- <p class="q-mb-sm"/> -->
          <div class="col-12 q-pa-sm">
            <q-input 
            filled 
            v-model="info.user_data.company" 
            type="text" 
            label="單位名稱"
            hint="你的單位名稱"
            lazy-rules :rules="[ val => verify_columns('company', val)]">
              <template v-slot:prepend>
                <q-icon name="apartment" />
              </template>
              <!-- <template v-slot:append>
                <q-icon name="close" @click="info.user_data.email = ''" class="cursor-pointer" />
              </template> -->
            </q-input>
          </div>
          <!-- <p class="q-mb-sm"/> -->
        </div>

        <div class ="justify-center col-12 q-pa-sm" align="center">
          <q-uploader
            style="width:auto;"
            :factory="uploadFile"
            @added="check_if_added"
            hide-upload-btn
            color="white"
            text-color="grey-6"
            label=""
            max-file-size="1536000"
            accept=".jpg, image/png, .jpeg"
            ref="uploader"
            @rejected="onRejected()"
          >

            <template v-slot:header="scope">
              <div class="row no-wrap items-center q-pa-sm q-gutter-xs">
                <!-- <q-btn v-if="scope.queuedFiles.length > 0" icon="clear_all" @click="scope.removeQueuedFiles" round dense flat >
                  <q-tooltip>清除</q-tooltip>
                </q-btn> -->

                <q-btn :ripple="false" :disabled="!scope.canAddFiles"  type="a" icon="add_box" round dense flat>
                  <q-uploader-add-trigger v-if="scope.canAddFiles" />
                  <q-tooltip>選擇圖片</q-tooltip>
                </q-btn>

                <q-spinner v-if="scope.isUploading" class="q-uploader__spinner" />
                <div class="col">
                  <div class="q-uploader__title">更改Logo圖片，檔案請小於1MB</div>
                  <div class="q-uploader__subtitle"> 檔案大小：{{ scope.uploadSizeLabel }}</div>
                </div>

                <q-btn
                  :ripple="false"
                  :disabled="scope.canAddFiles"
                  size="12px"
                  flat
                  dense
                  round
                  icon="delete"
                  @click="scope.removeQueuedFiles"
                >
                  <q-tooltip>清除圖片</q-tooltip>
                </q-btn>

              </div>
            </template>

            <template v-slot:list="scope">
              <q-list separator>
                <q-item v-for="file in scope.files" :key="file.name">
                  <q-item-section
                    top side
                    :key="file.name"
                    v-if="file.__img"
                    
                  >
                    <img :src="file.__img.src" style="width:100%; height:auto;">
                  </q-item-section>
                </q-item>
              </q-list>
            </template>

          </q-uploader>
        </div>


        <q-btn :loading="update_loading" color="primary" type="submit" label="更新" style="margin-top: 30px;">
          <template v-slot:loading>
            <q-spinner-hourglass class="on-center" />
          </template>
        </q-btn>

    </q-form>
  </div>

</div>
</template>

<script>
import axios from 'axios'
import { openURL } from 'quasar'
import md5 from 'crypto-js/md5';
import TwCitySelector from 'tw-city-selector';

var check_flag = [false, false, false, false, false, false, false, false, false]

export default {
  name: 'device',
  data () {
    return {
      info: {
        user_data:{
          email: '',
          password: '',
          first_name: '',
          state: '',
          postcode: '',
          address_1: '',
          phone: '',
          company: '',
          logo_base64: '',
        }
      },
      ifttt: {},
      update_data:{},
      original_password: '',
      isPwd: true,
      input_fail: false,
      update_fail: false,
      update_loading: false,
      login_status: '',
      role: '',
      vertification_token: '',
      user_devices_list: [],
      login_timestamp: 0,
      county: '',
      district: '',
      logo_base64: '',
      new_upload_logo: '',
      new_upload_logo_base64: '',
      check_if_added_time: 0,
    }
  },
  created () {
  },
  mqtt:{
    'frrut/pe100/account_update/receive_data/#' (data, topic) {
      // console.log('topic: ', topic)
      if (topic === ('frrut/pe100/account_update/receive_data/' + this.vertification_token)) {
        this.dataobj = JSON.parse(data)
        console.log('this.dataobj: ', this.dataobj)

        if(this.dataobj === true){
          localStorage.removeItem('loginInfo')
 
          var new_loginInfo = {
            'user_info':{
              'email': this.info.user_data.email,
              'password': this.update_data.user_data.password,
              'first_name': this.info.user_data.first_name,
              'state': this.info.user_data.state,
              'postcode': this.info.user_data.postcode,
              'address_1': this.info.user_data.address_1,
              'phone': this.info.user_data.phone,
              'company': this.info.user_data.company,
              'logo_base64': this.info.user_data.logo_base64
            },
            'login_status': this.login_status, 
            'role': this.role,
            'vertification_token': this.vertification_token, 
            'login_timestamp': this.login_timestamp,
            'user_devices_list': this.user_devices_list,
            'ifttt': this.ifttt,
          }

          localStorage.setItem('loginInfo', JSON.stringify(new_loginInfo))

          this.$mqtt.unsubscribe('frrut/pe100/account_update/receive_data/#')

          window.location.reload()
          // this.update_loading = false
        }
        else{
          console.log('something wrong.')
        }
      } 

    },
  },
  mounted () {
    // get login user information
    var loginInfo = JSON.parse(localStorage.getItem('loginInfo'))
    // console.log('loginInfo: ', loginInfo)
    this.login_status = loginInfo.login_status
    this.role = loginInfo.role
    this.vertification_token = loginInfo.vertification_token
    this.login_timestamp = loginInfo.login_timestamp
    this.user_devices_list = loginInfo.user_devices_list
    this.logo_base64 = loginInfo.user_info.logo_base64
    this.ifttt= loginInfo.ifttt

    this.first_load_logo()

    // 隱藏原始密碼
    this.info.user_data = loginInfo.user_info
    this.original_password = this.info.user_data.password
    this.info.user_data.password = '********'
    // console.log("this.original_password: ", this.original_password)

    this.county = this.info.user_data.state.slice(0, 3)
    this.district = this.info.user_data.state.slice(3)

    // 縣市、區域、郵遞區號
    new TwCitySelector({
      el: '.city-selector-set',
      elCounty: '.county', // 在 el 裡查找 element
      elDistrict: '.district', // 在 el 裡查找 element
      elZipcode: '.zipcode', // 在 el 裡查找 element
      countyValue: this.county,
      districtValue: this.district,
      zipcodeValue: this.info.user_data.postcode,
      standardWords: true,
    });

  },
  methods: {
    singleFileToBase64(files, reader) {
      reader = new FileReader();   
      // read the file into a base64 format 
      reader.readAsDataURL(files);

      return new Promise((resolve, reject) => {
          reader.onerror = () => {
              reader.abort();
              reject("Insert error message here")
          };

          // return the base 64 string
          reader.onload = function () {
              resolve(reader.result);
          };
      })
    },
    close_update_fail(){
      this.update_fail = false
    },
    check_if_added(){
      this.check_if_added_time += 1
      // this.new_upload_logo = this.$refs.uploader.files[0]
      // console.log("this.$refs.uploader: ", this.$refs.uploader)
      // console.log('this.new_upload_logo: ', this.new_upload_logo)

      let upload_file = this.singleFileToBase64(this.$refs.uploader.files[0]).then(data => {
        // console.log("data: ", data)
        this.new_upload_logo_base64 = data
        // console.log("this.new_upload_logo_base64: ", this.new_upload_logo_base64)
      }).catch(() => {
        this.$q.notify({
          color: 'negative',
          message: 'Failed to convert file...'
        })
        reject()
      })
    },
    first_load_logo(){
      function dataURLtoFile(dataurl, filename) {//將base64轉換為檔案
        var arr = dataurl.split(','), 
            mime = arr[0].match(/:(.*?);/)[1],
            bstr = atob(arr[1]), 
            n = bstr.length, 
            u8arr = new Uint8Array(n);
        while(n--){
            u8arr[n] = bstr.charCodeAt(n);
        }
        return new File([u8arr], filename, {type:mime});
      }   

      var file_type = this.logo_base64.split(',')[0].match(/:(.*?);/)[1].split('/')[1]
      // console.log("file_type: ", file_type)
      var file_logo = dataURLtoFile(this.logo_base64, 'logo.' + file_type)
      this.$refs.uploader.addFiles([file_logo])
    },
    uploadFile(files){
      // files.push(this.info.user_data)
      console.log("files: ", files)
      
      return new Promise((resolve) => {
        // simulating a delay of 2 seconds
        setTimeout(() => {
          resolve({
            url: 'http://mk100.frrut.net:3000/pe100/customize_logo',
            // methods: 'POST',
            headers: [{ name: 'email', value: this.info.user_data.email}]
          })
        }, 1000)
      })
    },
    onRejected (rejectedEntries) {
      // Notify plugin needs to be installed
      // https://quasar.dev/quasar-plugins/notify#Installation
      this.$q.notify({
        type: 'negative',
        message: `此檔案不符上傳格式，請確認後再次上傳。`
      })
      check_flag[8] = false
    },
    updateProfile () {

      // console.log(this.$refs.uploader.files[0])
      this.update_loading = true

      this.county === ''?(check_flag[2] = false):(check_flag[2] = true)
      this.district === ''?(check_flag[3] = false):(check_flag[3] = true)
      this.info.user_data.postcode === ''?(check_flag[4] = false):(check_flag[4] = true)

      if(this.$refs.uploader.files[0]){
        check_flag[8] = true
        console.log("have file now!")
        this.info.user_data.logo_base64 = this.new_upload_logo_base64
      }
      else{
        console.log("no file!")
        this.info.user_data.logo_base64 = ''
        check_flag[8] = false
      }

      let checker = arr => arr.every(v => v === true);
      if(checker(check_flag)){


        this.update_data = {
          user_data: JSON.parse(JSON.stringify(this.info.user_data)),
          vertification_token: this.vertification_token,
          user_devices_list: this.user_devices_list
        }
        // console.log("update_data: ", this.update_data)

        if(this.info.user_data.password === "********"){
          this.update_data.user_data.password = this.original_password
        }
        else{
          this.update_data.user_data.password = md5(this.update_data.user_data.password).toString()
        }
        
        // this.update_data.user_data.logo_base64 = this.new_upload_logo_base64

        this.update_data.user_data.state = this.county + this.district


        // 判斷是否有更新logo
        if(this.check_if_added_time === 1){
          console.log("no logo change")
          this.update_data.user_data.logo_base64 = ''
        }
        else{
          console.log("update logo!!!")
          // this.$refs.uploader.upload()
        }

        this.$mqtt.subscribe('frrut/pe100/account_update/receive_data/#')
        this.$mqtt.publish('frrut/pe100/account_update', JSON.stringify(this.update_data))

        setTimeout(() => {
          this.update_fail = true
          this.update_loading = false
        }, 30000)
      }
      else{
        this.input_fail = true
        this.update_loading = false
        return false
      }
    },
    verify_columns(which, val){
      switch(which)
      {
        case 'password':
          return new Promise((resolve, reject) => {
            if(!val){
              resolve(!!val || '此欄位不得為空')
              check_flag[0] = false}
            // else if(val.match(/\W/)){
            //   // console.log(val.match(/\W/))
            //   resolve(!val.match(/\W/) || '密碼不得包含星號、冒號、引號、空白等特殊字元')
            //   check_flag[0] = false
            // }
            else if (val.length < 6){
              resolve(val.length >= 6 || '密碼至少需 6 位以上')
              check_flag[0] = false
            }            
            else{
              resolve(val=> val || '')
              check_flag[0] = true}
          })
          break;

        case 'first_name':
          return new Promise((resolve, reject) => {
            // console.log(val, ', length: ', val.length)
            // console.log("after function: ", val.replace(/[^\x00-\xff]/g,"xx").length)
            if(!val){
              resolve(!!val || '此欄位不得為空')
              check_flag[1] = false}
            else if(val.replace(/[^\x00-\xff]/g,"xx").length > 14){
              resolve(val.replace(/[^\x00-\xff]/g,"xx").length < 14 || '中英文合計最多 14 位元。（中文：2 位元，英文：1 位元）')
              check_flag[1] = false}
            else{
              resolve(val=> val || '')
              check_flag[1] = true}
            })
          break;

        case 'address_1':
          return new Promise((resolve, reject) => {
            if(!val){
              resolve(!!val || '此欄位不得為空')
              check_flag[5] = false}
            else{
              resolve(val=> val || '')
              check_flag[5] = true}
            })
          break;

        case 'phone':
          return new Promise((resolve, reject) => {
            if(!val){
              resolve(!!val || '此欄位不得為空')
              check_flag[6] = false}
            else if(val.length != 12){
              resolve(val.length == 12 || '格式錯誤')
              check_flag[6] = false}
            else{
              resolve(val=> val || '')
              check_flag[6] = true}
          })
          break;

        case 'company':
          return new Promise((resolve, reject) => {
            if(!val){
              resolve(!!val || '此欄位不得為空')
              check_flag[7] = false
              }
            else{
              resolve(val=> val || '')
              check_flag[7] = true
              }})
          break;
        
        default:
          alert("未知錯誤發生。");
      }
    },
    // reloadPage () {
    //   window.location.reload()
    // },
    // resetSlack () {
    //   console.log(this.info.data)
    //   axios.put(baseURL + this.loginInfo.id,
    //     { 'slack_user_id': '', 'slack_access_token': '', 'slack_team_id': '', 'webhook_channel_id': '', 'webhook_configuration_url': '', 'webhook_channel_url': '', 'webhook_channel': '' })
    //     .then(response => {
    //       this.info = response
    //       console.log(this.info.data)
    //     })
    //     .catch(error => {
    //       this.$router.push('/')
    //       console.log(error)
    //     })
    // },
    // slackURL(){
    //   var loginInfo_slack = JSON.parse(localStorage.getItem('loginInfo') || '{}')
    //   var slackURL = 'https://slack.com/oauth/authorize?scope=incoming-webhook&client_id=914264040262.899946458546&state=' + loginInfo_slack.id
    //   console.log(slackURL)
    //   openURL(slackURL)
    // }
  }
}
</script>

<style>
  .city-selector-set select, input {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    /* border-color: #6ec3f5; */
    border-width: 1px;
    border-radius: 0.5em;
    /* color: #1390e8; */
    margin-right: 10px;
    margin-top: 10px;
    margin-bottom: 10px;
    outline: none;
    padding: .3em 1em;
  }

</style>
