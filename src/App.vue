<script setup>
import { googleSdkLoaded } from 'vue3-google-login'
import axios from 'axios'
import { ref } from 'vue'

const userData = ref({})
const secret_key = import.meta.env.VITE_SECRET_KEY;
const client_id = import.meta.env.VITE_CLIENT_ID;
const redirect_uri = import.meta.env.VITE_REDIRECT_URL;

const signInWithGoogle = () => {
  googleSdkLoaded(google => {
    google.accounts.oauth2
      .initCodeClient({
        client_id: client_id,
        scope: 'email profile openid',
        redirect_uri: redirect_uri,
        callback: response => {
          if (response.code)
            fetchUserDataFrom(response.code)
        },
      })
      .requestCode()
  })
}

const  fetchUserDataFrom = async (code) => {
    try {
      localStorage.setItem('gCode', JSON.stringify(code))

      const { data } = await axios.post(
        'https://oauth2.googleapis.com/token',
        {
          code,
          client_id: client_id,
          client_secret: secret_key,
          redirect_uri: redirect_uri,
          grant_type: 'authorization_code',
        },
      )

      if (data) {
        const accessToken = data.access_token

        // Fetch user details using the access token
        const userObj = await axios.get(
          'https://www.googleapis.com/oauth2/v3/userinfo',
          {
            headers: {
              Authorization: `Bearer ${accessToken}`,
            },
          },
        )

        if (userObj && userObj.data) {
          // save copy in storage
          localStorage.setItem('user', JSON.stringify(userObj.data))
          userData.value = userObj.data
        }
        else {
          // Handle the case where userResponse or userResponse.data is undefined
          console.error('Failed to fetch user data')
        }
      }
    }
    catch (e) {
      console.error('Failed to exchaange token', e)
    }
  }

</script>

<template>
  <div class="">
    <button @click="signInWithGoogle">Continue with google</button>
    <p>{{ userData }}</p>
  </div>
</template>

<style scoped>

</style>
