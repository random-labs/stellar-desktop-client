/**
 * 顶部工具栏
 */
<template>
  <div class="cp-toolbar">

    <v-toolbar :height="`60px`" fixed dark :class="color + ' itoolbar'" dense :clipped-left='true'>
      <v-btn icon v-if="showbackicon" @click="back" class="white--text">
            <v-icon class="back-icon font28">&#xE5CB;</v-icon>
      </v-btn>
      <slot name="left-tool">
        <v-btn icon style="visibility:hidden;" v-if="!showmenuicon && !showbackicon">
          <v-icon class="back-icon"></v-icon>
        </v-btn>
      </slot>
      <v-toolbar-title>{{$t('AppName')}}</v-toolbar-title>
      
      <v-spacer/>
      <div class="flex-row ml-4 mr-2">
        <div class="account" @click="toMy">{{selectedAccount.name || account.name}}</div>
        <div class="mbtn iconbtn" flat  @click="toMy"><v-icon>arrow_drop_down</v-icon></div>
        <div>&nbsp;&nbsp;</div>
        <div class="mbtn iconbtn" flat @click="doLock"><v-icon>lock</v-icon></div>
      </div>


    </v-toolbar>


    <v-dialog persistent v-model="showPwdSheet" v-if="showPwdSheet" dark max-width="460px">
      <div class="sheet-content">
        <div class="sheet-title">
          <h4 class="title" v-if="!lockpass">
            <slot name='switch_password'> 
          <!--if call for passowrd -->
              <span>{{$t('ChangeAccount')}}</span>
            </slot>
          </h4>
        </div>
        <div class="sheet-title">
          <div class="label">{{$t('Account.AccountName')}}</div>
          <div class="value">{{selectedAccount.name || account.name}}</div>
        </div>
        <div class="sheet-input">
          <v-text-field
                name="password"
                :label="$t('Account.Password')"
                v-model="password"
                :append-icon="pwdvisible ? 'visibility' : 'visibility_off'"
                :append-icon-cb="() => (pwdvisible = !pwdvisible)"
                :type="pwdvisible ? 'text':'password'"
                required dark
                @keyup.enter.native="okPwdInput"
              ></v-text-field>
        </div>
        <div  class="sheet-btns">
          <div class="sheet-btn" v-if="!lockpass" @click="canclePwdInput">{{$t('Button.Cancel')}}</div>
          <div class="sheet-btn" v-else @click="resetPwdInput">{{$t('Reset')}}</div>
          <div class="sheet-btn" @click="okPwdInput">{{$t('Button.OK')}}</div>
        </div>
      </div>
    </v-dialog>

  </div>
</template>

<script>
import { mapState, mapActions} from 'vuex'
export default {
  data(){
    return {
      showmenu: false,
      selected: {},
      password: null,
      showaccounts: false,
      selectedAccount: {},
      showPwdSheet: false,
      selectedIndex:null,
      password: null,
      pwdvisible: false,
      checkPwd: false,//是否正在校验密码

      selectedMenuIndex:null,
      pageNames:[
        'MyAssets',
        'History',
        'TradeCenter',
        'Apps',
        'Funding',
        'MySettings',
        'SettingsParent'
      ]

    }
  },
  computed:{
    ...mapState({
      accounts: state => state.accounts.data,
      account: state => state.accounts.selectedAccount,
      selectedAccountIndex: state => state.accounts.selected,
      accountData: state => state.accounts.accountData,
      app: state => state.app,
    }),
  },
  props: {
    title: {
      type: String,
      default: ''
    },
    showbackicon: {
      type: Boolean,
      default: true
    },
    showmenuicon: {
      type: Boolean,
      default: false
    },
    color: {
      type: String,
      default: 'secondarygray'
    },
    shadow: {
      type: Boolean,
      default: false
    },
    //是否锁定密码输入窗口，要求必须输入密码
    lockpass:{
      type: Boolean,
      default: false
    },
    menuName:{
      type: String
    }
  },
  beforeMount(){
    if(this.menuName){
      this.selectedMenuIndex = this.pageNames.indexOf(this.menuName)
    }else{
      this.selectedMenuIndex = 0
    }
  },
  methods: {
    ...mapActions({
      cleanAccount:'cleanAccount',
      choseAccount: 'changeAccount',
      choseAccountNoPwd: 'changeAccountNoPassword',
      getAccountInfo: 'getAccountInfo'

    }),
    changetheme(color){
      if (color=== 'ui'){
        this.$vuetify.theme.primary = '#21ce90'
      }else{
        this.$vuetify.theme.primary = 'red'
      }
    },
    redirect(url){
      this.$router.push(url)
    },
    changeAccount(index,item){
      this.selectedAccount = item
      this.selectedIndex = index
      this.showPwdSheet = true
    },
    resetPwdInput(){
      this.password = null
    },
    //取消输入密码，则直接无密码跳转账户
    canclePwdInput(){
      this.showPwdSheet=false
      //this.choseAccountNoPwd({index:this.selectedIndex, account: this.selectedAccount.address})
    },
    //校验密码是否正确，不正确则提示，正确则跳转账户
    okPwdInput(){
      if(this.checkPwd)return
      //无密码则报错
      if(!this.password){
        this.$toasted.error(this.$t('Error.PasswordWrong')) //请输入密码
        return
      }
      //检验密码
      this.checkPwd = true
      let data = {
        index: this.selectedIndex === null ? this.selectedAccountIndex : this.selectedIndex,
        address: this.selectedAccount.address || this.account.address,
        password: this.password
      }
      console.log(data)
      this.choseAccount(data).then(account=>{
        this.$toasted.show(this.$t('Info.ChangeAccountSuccess'));
        this.showPwdSheet = false;
        this.checkPwd = false;
        this.password = null;
        this.showaccounts = false;
        this.getAccountInfo(this.account.address)
      }).catch(err=>{
        console.error('change account error')
        console.error(err)
        this.$toasted.error(this.$t('Error.PasswordWrong'))
        this.checkPwd = false
      })
    },
    isPage(path){
      let url = this.$route.name
      if(url.indexOf(path.name)===0){
        return ' menuactive'
      }
      return ''
    },
    showMenu(){
      this.showmenu = !this.showmenu
    },
    switchShowAccounts(){
      this.showaccounts = !this.showaccounts
    },
    back(){
      //console.log('----on click -- go back')
      this.$router.back()
      //this.$emit('goback');
    },
    changeaccount(index,item){
      //console.log('------change account----')
      //if(item.address === this.account.address){
      //  return;
      //}
      // 弹出密码对话框，
      this.showmenu = !this.showmenu
      if(this.changeAccount){
        this.changeAccount(index,item)
      }
    },
    showPasswordLogin(){
      this.showPwdSheet = true
    },
    //取消输入密码，则直接无密码跳转账户
    canclePwdInput(){
      this.showPwdSheet=false
      //this.choseAccountNoPwd({index:this.selectedIndex, account: this.selectedAccount.address})
    },
    //校验密码是否正确，不正确则提示，正确则跳转账户
    okPwdInput(){
      if(this.checkPwd)return
      //无密码则报错
      if(!this.password){ 
        this.$toasted.error(this.$t('Error.PasswordWrong')) //请输入密码
        return
      }
      //检验密码
      this.checkPwd = true
      let data = {
        index: this.selectedIndex === null ? this.selectedAccountIndex : this.selectedIndex,
        address: this.selectedAccount.address || this.account.address,
        password: this.password
      }
      console.log(data)
      this.choseAccount(data).then(account=>{
        this.$toasted.show(this.$t('Info.ChangeAccountSuccess'))
        this.showPwdSheet = false
        this.checkPwd = false
        this.password = null
        this.showaccounts = false
        
        this.getAccountInfo(this.account.address)

      }).catch(err=>{
        console.error('change account error')
        console.error(err)
        this.$toasted.error(this.$t('Error.PasswordWrong'))
        this.checkPwd = false
      })
    },
    toPage(index){
      this.selectedMenuIndex = index
      this.$router.push({name: this.pageNames[index]})
    },
    toMyAssets(){
      this.selectedMenuIndex = 0
      this.$router.push({name: 'MyAssets'})
    },
    toTradeCenter(){
      this.selectedMenuIndex = 1
      this.$router.push({name: 'TradeCenter'})
    },
    toDapps(){
      this.selectedMenuIndex = 2
      this.$router.push({name: 'Apps'})
    },
    toFunding(){
      this.selectedMenuIndex = 3
      this.$router.push({name: 'Funding'})
    },
    toMy(){
      this.selectedMenuIndex = 4
      this.$router.push({name: 'MySettings'})
    },
    toSettings(){
      this.selectedMenuIndex = 5
      this.$router.push({name: 'SettingsParent'})
    },
    doLock(){
      //判断是否开启了锁屏，未开启则启动锁屏
      if(this.app.enablePin){
        this.$router.push({name: 'PinLock'})
      }else{
        this.$router.push({name: 'SetPinCode'})
      }

    }
  }
}
</script>

<style lang="stylus" scoped>

@require '../stylus/color.styl'

.cp-toolbar
  z-index: 99
.menu-icon
  color: $primarycolor.font
.back-icon
  font-size: 36px
.toolbar-title
  text-align: center
  width: 100%
  flex:1

.sheet-content
  background: $secondarycolor.gray
  color: $primarycolor.font
  padding: 10px 10px
  word-wrap: break-word
  .sheet-title
    .title
      height: 40px
      line-height: 40px
      font-size: 32px
      text-align: center
      color: $primarycolor.green
      padding: 10px 10px
    .label
      color: $secondarycolor.font
    .value
      font-size: 16px
  .sheet-btns
    margin-top: 10px
    display: inline-block
    width: 100%
    padding-bottom: 0
    padding-bottom: constant(safe-area-inset-bottom)
    padding-bottom: env(safe-area-inset-bottom)
    .sheet-btn
      float: left
      width: 50%
      height: 40px
      line-height: 40px
      text-align: center
      font-size: 16px
      color: $primarycolor.green

.mactive
  background: $secondarycolor.green
  // color: $primarycolor.green
.mbtn
  line-height: 48px
  padding: 0 10px!important
  cursor: pointer
  white-space: nowrap
  
.itoolbar
  height: 60px
  line-height: 60px
.iconbtn
  padding-top: 5px!important
.account
  cursor: pointer
</style>
