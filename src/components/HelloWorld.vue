<script setup lang="ts">
import { reactive, ref } from "vue";
import axios from "axios";
import { Device } from "@twilio/voice-sdk";

defineProps<{ msg: string }>();

const filters = reactive({
  baseUrl: "https://1162-165-16-112-159.ngrok-free.app",

  phone: "+218920920110",
  password: "password",

  personToCallName: "",
});

const myName = ref("");
const token = ref(localStorage.getItem("token") || "");
const twilioToken = ref("");

const login = async () => {
  try {
    const res = await axios.post(`${filters.baseUrl}/api/v1/user/login`, {
      phone: filters.phone,
      password: filters.password,
    });
    localStorage.setItem("token", res.data.token);

    token.value = res.data.token;
    twilioToken.value = JSON.parse(res.data.twilio_token).token;
  } catch (error) {
    console.log(error);
  }
};

const callPerson = async () => {
  console.log(token.value);
  const res = await axios.post(
    `${filters.baseUrl}/api/v1/general/client-id`,
    null,
    {
      headers: {
        Authorization: `Bearer ${token.value}`,
      },
    }
  );

  const callResponse = await axios.post(
    `${filters.baseUrl}/api/v1/general/make-call-by-client`,
    null,
    {
      headers: {
        Authorization: `Bearer ${token.value}`,
      },
    }
  );
  console.log(callResponse);

  myName.value = res.data;

  const device = new Device(twilioToken.value, {
    debug: true,
    codecPreferences: ["opus", "pcmu"],
  });

  // Add event listeners for device events
  device.on("ready", () => {
    console.log("Device is ready");
  });

  device.on("error", (error: any) => {
    console.error("Error:", error);
  });

  device.on("incoming", (connection: any) => {
    console.log("Incoming connection:", connection);
  });

  // Make a call
  const params = {
    To: "client:" + filters.personToCallName,
    // From: myName.value,
    // Add any additional parameters as needed
  };
  const connection = device.connect(params);
};
</script>

<template>
  <div style="margin-bottom: 5rem">
    <h1>login</h1>
    <div>
      <label>url</label>
      <br />
      <input type="text" v-model="filters.baseUrl" />
    </div>
    <div>
      <label>phone</label>
      <br />
      <input type="text" v-model="filters.phone" />
    </div>
    <div>
      <label>password</label>
      <br />
      <input type="text" v-model="filters.password" />
      <br />
      <button style="margin-top: 1rem" @click="login">login</button>
    </div>
  </div>
  <div>
    <h1>call</h1>
    <label>person to call</label>
    <input type="text" v-model="filters.personToCallName" />
    <button @click="callPerson">call</button>
  </div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
