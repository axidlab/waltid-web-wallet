<template>
  <div>
    <div id="content-credentials" v-if="issuanceSessionInfo">
      <h2>{{$t('ISSUANCE_INITIATION.TITLE')}}</h2>
      <em>{{$t('ISSUANCE_INITIATION.SUBTITLE')}}</em>
      <div class="_scrollable _container d-flex flex-column align-items-center justify-content-center">
        <div class="mb-2">
          <b>{{$t('CREDENTIAL.ISSUER')}}:</b><br/>
          {{issuanceSessionInfo.issuerId}}
        </div>
        <div class="mb-2">
          <b>{{$t('ISSUANCE_INITIATION.OFFERED_CREDENTIALS')}}:</b>
          <div v-for="credentialType in issuanceSessionInfo.credentialTypes" :key="credentialType">
            {{credentialType}}
          </div>
        </div>
        <div class="mb-2">
          <b>{{$t('ISSUANCE_INITIATION.SUBJECT_DID')}}:</b>
          <div class="text-truncate" style="max-width: 20em;" data-bs-toggle="tooltip" data-bs-placement="bottom" :title="currentDid">{{currentDid}}</div>
        </div>
        <div class="mb-2" v-if="issuanceSessionInfo.userPinRequired">
          <b>{{$t('ISSUANCE_INITIATION.USER_PIN')}}:</b>
          <input type="password" class="form-control border-primary mb-2 w-50 mx-auto" placeholder="PIN" aria-label="PIN" v-model="userPin" autocomplete="new-password">
        </div>
        <div class="_button mt-2">
          <button href="#share" :disabled="issuanceSessionInfo.userPinRequired && userPin == null" class="btn btn-primary _share col-12 mb-2" @click="accept()">{{$t('RECEIVE_CREDENTIALS.ACCEPT')}}</button>
          <a href="#reject" class="_reject col-12">{{$t('RECEIVE_CREDENTIALS.REJECT')}}</a>
        </div>
      </div>
    </div>
  </div>
</template>
	
<script>
export default {
  name: "InitiateIssuance",
  data() {
    return {
      issuanceSessionInfo: null,
      userPin: null
    }
  },
  computed: {
    currentDid() {
      console.log(this.$store.state.wallet.currentDid)
      return this.$store.state.wallet.currentDid
    }
  },
  async asyncData ({ $axios, query }) {
    if(query.sessionId != null) {
      const issuanceSessionInfo = await $axios.$get("/api/wallet/issuance/info", { params: query })
      return { issuanceSessionInfo }
    }
  },
  methods: {
    accept: async function() {
      this.$axios.$get('/api/wallet/issuance/continueIssuerInitiatedIssuance', { params: {
        did: this.currentDid, sessionId: this.issuanceSessionInfo.id, userPin: this.userPin
      }}).then(
        location => {
          if(location.startsWith("/")) {
          this.$router.replace(location)
          } else {
            window.location = location
          }
        },
        reason => {
          this.$router.replace("/IssuanceError?reason="+reason)
        }
      )
      
    }
  }
	
}
</script>
	
<style scoped>
	
</style>