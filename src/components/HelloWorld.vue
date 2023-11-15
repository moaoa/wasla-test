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
const twilioToken = ref(localStorage.getItem("twilioToken") || "");

let device: any;

const login = async () => {
  try {
    const res = await axios.post(`${filters.baseUrl}/api/v1/user/login`, {
      phone: filters.phone,
      password: filters.password,
    });
    localStorage.setItem("token", res.data.token);
    localStorage.setItem("twilioToken", res.data.twilio_token);

    token.value = res.data.token;
    twilioToken.value = res.data.twilio_token;

    device = new Device(twilioToken.value);

    device.on("ready", () => {
      console.log("Device is ready");
    });

    device.on("error", (error: any) => {
      console.error("Error:", error);
    });

    device.on("incoming", (connection: any) => {
      console.log("Incoming connection:", connection);
    });

    const res2 = await axios.post(
      `${filters.baseUrl}/api/v1/general/client-id`,
      null,
      {
        headers: {
          Authorization: `Bearer ${token.value}`,
        },
      }
    );
    myName.value = res.data;
    console.log("me: ", res2.data);

    console.log(res.data);
  } catch (error) {
    console.log(error);
  }
};

const callClient = async () => {
  console.log(token.value);

  const callResponse = await axios.post(
    `${filters.baseUrl}/api/v1/general/make-call-by-client`,
    {
      to: filters.personToCallName,
    },
    {
      headers: {
        Authorization: `Bearer ${token.value}`,
      },
    }
  );
  console.log(callResponse);

  // Add event listeners for device events

  // Make a call
  const params = {
    // To: "client:" + filters.personToCallName,
    To: filters.personToCallName,
  };

  console.log("calling: ", device);
  console.log(params);
  device.connect(params);
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
      <select v-model="filters.phone">
        <option value="+218920920110">+218920920110</option>
        <option value="+218912245321">+218912245321</option>
      </select>
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
    <button @click="callClient">call</button>
  </div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
