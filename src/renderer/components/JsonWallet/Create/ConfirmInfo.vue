<template>
  <div class="container json-confirm-container">
    <p><b>{{$t('createJsonWallet.labelN')}}: </b> {{label}}</p>
    <p><b>{{$t('createJsonWallet.addressN')}}: </b> {{ address }}</p>
    <p><b>{{$t('createJsonWallet.pubKeyN')}}: </b> {{publicKey}}</p>
    <p><b>{{$t('createJsonWallet.signatureSchemeN')}}: </b> SHA256withECDSA </p>
    <p><b>{{$t('createJsonWallet.wif')}}: </b> {{wif}}</p>

    <div class="backup-text">
      <p class="font-medium-black">
        <span></span>
         <a-icon type="warning" /> {{$t('createJsonWallet.backupWallet')}}</p>
         <a-button type="primary" @click="downloadWallet">{{$t('createJsonWallet.download')}}</a-button>
    </div>
    <div class="confirm-btns">
      <div class="confirm-btn-container">
        <a-button type="default" class="btn-cancel" @click="back">{{ $t('createJsonWallet.back') }}</a-button>
        <a-button type="primary" class="btn-next" @click="next">{{ $t('createJsonWallet.next') }}</a-button>
      </div>
    </div>
  </div>
</template>

<script>
  import {mapState} from 'vuex'
  import {Crypto, Wallet, Account} from 'ontology-ts-sdk'
  import FileHelper from "../../../../core/fileHelper"
  import dbService from '../../../../core/dbService'
  import {WALLET_TYPE,DEFAULT_SCRYPT} from '../../../../core/consts'
  import en from '../../../../common/lang/en'
  import zh from '../../../../common/lang/zh'

  export default {
    name: 'ConfirmInfo',
    data() {
      const langType = localStorage.getItem('user_lang') || 'en';
      const lang = langType === 'en' ? en : zh;
      return {
        lang: lang,
        processing: false
      }
    },
    mounted() {
      this.downloadWallet()
    },
    beforeDestroy() {
      console.log('clear')
      this.$store.commit('INIT_JSON_WALLET')
    },
    computed: {
      ...mapState({
        label: state => state.CreateJsonWallet.label,
        account: state => state.CreateJsonWallet.account,
        downloadContent: state => state.CreateJsonWallet.downloadContent,
        address: state => state.CreateJsonWallet.address,
        publicKey: state => state.CreateJsonWallet.publicKey,
        wif: state => state.CreateJsonWallet.wif
      })
    },
    methods: {
      back() {
        this.$store.commit('SUB_CREATE_JSON_STEP')
      },
      downloadWallet() {
        const commonWallet = this.account
        let wallet = Wallet.create(commonWallet.label || "")
        console.log(wallet)
        wallet.scrypt.n = 16384;
        const account = Account.parseJsonObj(commonWallet)
        wallet.addAccount(account)
        FileHelper.downloadFile(wallet.toJsonObj(), commonWallet.label);
      },
      next() {
        this.$store.dispatch('showLoadingModals')

        

        //Download file
        // FileHelper.downloadFile(this.downloadContent)

        //save to db
        const wallet = {
          type : WALLET_TYPE.CommonWallet,
          address: this.address,
          wallet: this.account
        }
        dbService.insert(wallet, function (err, newDoc) {
          if (err) {
            console.log(err)
          }
          // console.log(newDoc)
        })

        this.$store.commit('INIT_JSON_WALLET')
        this.$message.success(this.$t('createJsonWallet.createSuccess'))
        this.$router.push({name: 'Wallets'})
      }
    }
  }
</script>

<style scoped>
  .json-confirm-container {
    width: 36rem;
    padding: 15px;
    border:1px solid #dddddd;
  }

  .confirm-btns {
    position: fixed;
    bottom: 0;
    width: calc(100% - 4rem);
    height: 85px;
    left: 4rem;
    background: #FFFFFF;
    box-shadow: 0 -1px 6px 0 #F2F2F2;
    z-index: 1000;
  }

  .confirm-btn-container {
    width: 540px;
    margin: 20px auto;
  }

  .confirm-btn-container :last-child {
    float: right;
  }
  .backup-text {
    text-align: center;
    /* padding: 20px; */
    /* border: 1px solid #dddddd; */
    font-size: 16px;
  }
  .backup-text p {
    margin-bottom: 15px;
    font-size:16px !important;
  }

</style>
