<script setup lang="ts">
import axios from "axios";
import { Device } from "@twilio/voice-sdk";
import { useLocalStorage } from "@vueuse/core";

defineProps<{ msg: string }>();

const filters = useLocalStorage("filters", {
  baseUrl: "https://1162-165-16-112-159.ngrok-free.app",

  phone: "+218920920110",
  password: "password",

  personToCallName: "",
});

const myName = useLocalStorage("myName", "");
const token = useLocalStorage("token", "");
const twilioToken = useLocalStorage("twilioToken", "");

let device: any;

const login = async () => {
  try {
    const res = await axios.post(`${filters.value.baseUrl}/api/v1/user/login`, {
      phone: filters.value.phone,
      password: filters.value.password,
    });

    token.value = res.data.token;

    twilioToken.value = await getTwilioToken(res.data.token);

    device = new Device(twilioToken.value, { logLevel: "debug" });

    device.on(Device.EventName.Registered, () => {
      console.log("Device is registered");
    });

    device.on(Device.EventName.Error, (error: any) => {
      console.error("Error:", error);
    });

    device.on(Device.EventName.Incoming, (connection: any) => {
      console.log("Incoming connection:", connection);
    });

    device.on(Device.EventName.TokenWillExpire, async () => {
      // const newToken = await getAccessToken();
      // device.updateToken(newToken);
      console.log("token will expire");
    });

    const res2 = await axios.post(
      `${filters.value.baseUrl}/api/v1/general/client-id`,
      null,
      {
        headers: {
          Authorization: `Bearer ${token.value}`,
        },
      }
    );

    myName.value = res2.data;
  } catch (error) {
    console.log(error);
  }
};

const getTwilioToken = async (token: string) => {
  try {
    const res = await axios.post(
      `${filters.value.baseUrl}/api/v1/general/generate-twilioToken`,
      null,
      {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      }
    );
    console.log(res.data);
    return res.data.twilio_token;
  } catch (error) {
    console.log(error);
  }
};

const callClient = async () => {
  console.log(token.value);

  const callResponse = await axios.post(
    `${filters.value.baseUrl}/api/v1/general/make-call-by-client?to=${filters.value.personToCallName}`,
    null,
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
    to: filters.value.personToCallName,
    From: myName.value,
  };

  console.log("calling: ", device);
  console.log(params);

  const connection = device.connect(params).then((res: any) => {
    console.log("from promise: ", res);
  });

  console.log(connection);

  // connection.on(Device.EventName.Incoming, () => {
  //   console.log("incoming from connection");
  // });

  // connection.on(Device.EventName.Error, () => {
  //   console.log("error from connection");
  // });
};
</script>

<template>
  <div style="margin-bottom: 5rem">
    <h1>login</h1>
    <h1 style="color: red">{{ myName }}</h1>
    <div>
      <label>url</label>
      <br />
      <input type="text" v-model="filters.baseUrl" />
    </div>
    <div>
      <label>phone</label>
      <br />
      <select v-model="filters.phone">
        <option value="+218920920110" style="padding: 16px">
          +218920920110
        </option>
        <option value="+218912245321" style="padding: 16px">
          +218912245321
        </option>
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
