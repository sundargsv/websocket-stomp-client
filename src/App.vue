<template>
  <div id="app">
    <v-app>
      <v-app-bar
      app
      color="primary"
      dark
      >
        <div class="d-flex align-center">
          <v-img
            alt="Vuetify Logo"
            class="shrink mr-2"
            contain
            src="@/assets/some-icon.png"
            transition="scale-transition"
            width="50"
          />
        </div>

        <v-toolbar-title class="mx-2">Web socket STOMP client</v-toolbar-title>
        
        <v-dialog v-model="dialog" persistent max-width="800px">
          <template v-slot:activator="{ on, attrs }">
            
            <v-btn class="mx-2" fab dark small color="cyan" v-bind="attrs" v-on="on">
              <v-icon dark>mdi-pencil</v-icon>
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">Websocket Connection</span>
            </v-card-title>

            <v-card-text>
              <v-text-field v-model="websocketURL" label="WS/ WSS URL" required></v-text-field>
              <v-text-field v-model="clientToken" label="Client-Token" required></v-text-field>
              <v-text-field v-model="authToken" label="Authorization" required></v-text-field>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="saveWsConfig">Update Config</v-btn>
              <v-btn color="blue darken-1" text @click="dialog = false">Cancel</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-spacer></v-spacer>
      </v-app-bar>

      <v-main>
        <v-container class="lighten-5" style="max-width: 1550px">
          <!-- Columns are always 50% wide, on mobile and desktop -->
          <v-row>
            <v-col cols="5">
              <v-card class="pa-2" tile>
                <v-text-field v-model="webSocketSubscribeEndpoint" label="Subscribe Endpoint.." required></v-text-field>

                <v-text-field v-model="webSocketSendEndpoint" label="Send Endpoint.." required></v-text-field>

                <v-text-field v-model="webSocketInitMessage" label="Send Initial Message"
                hint="Please make sure, its in the JSON format"
                ></v-text-field>

                <div>
                  <v-btn class="ma-2" color="primary" @click="connect">Connect</v-btn>
                  <v-btn class="ma-2" @click="disconnect">Disconnect</v-btn>
                </div>

                <v-textarea v-model="webSocketSendMessage" label="Send WS Message" 
                hint="Please make sure, its in the JSON format"
                ></v-textarea>

                <div>
                  <v-btn class="ma-2" color="primary" @click="sendWSMessage">Send</v-btn>
                  <v-btn class="ma-2" @click="clearSendWSMessage">Clear</v-btn>
                </div>

              </v-card>
            </v-col>
            <v-col cols="7">
              <v-card class="pa-2" outlined tile>
                <label>Received WS Message</label>
                <ul>
                  <li v-for="message in messages" v-bind:key="message">{{ message }}</li>
                </ul>
              </v-card>
            </v-col>
            
          </v-row>
        </v-container>
      </v-main>
    </v-app>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { Component } from 'vue-property-decorator';
import { WebsocketService } from './services/websocket-service';
import WebsocketConnector from './components/WebsocketConnector.vue';


@Component({})
export default class App extends Vue {
  private webSocketService?: WebsocketService;
  private messages: string[] = [];

  private webSocketSubscribeEndpoint = "";
  private webSocketSendEndpoint  = "";
  private webSocketInitMessage = "";
  private webSocketSendMessage = "";

  private dialog = false;
  private websocketURL = "";
  private clientToken = "";
  private authToken = "Bearer ";

  saveWsConfig(){
    console.log("Websocket config updated with new data...");
    this.dialog = false;
  }

  connect(){
    console.log("Connect event clicked...,");
    if(this.websocketURL && this.clientToken && this.authToken && this.webSocketSubscribeEndpoint 
        && this.webSocketSendEndpoint && this.webSocketInitMessage){
          this.webSocketService = WebsocketService.getInstance(this.websocketURL, this.clientToken, this.authToken);
    this.webSocketService.connect(
      () => {
        this.webSocketService &&
          this.webSocketService.subscribe(
            this.webSocketSubscribeEndpoint,
            message => {
              if (message.body) {
                console.log('Received WS message: ', message.body);
                this.messages.push(message.body);

              } else {
                console.warn('Received empty message', message);
              }
            }
          );

        // send initial message to get jobs
        this.sendMessage();
      },
      () => {
        console.warn("Websocket disconnected successfully..")
      },
      () => {
        console.warn("Websocket error..")
      }
    );
        
    }else{
      console.error("One of the required field is empty...");
    }
  }

  disconnect(){
    this.webSocketService && this.webSocketService.disconnect();
    console.log("Websocket disconnected successfully");
  }

  sendWSMessage(){
    console.log("Send WS Message", this.webSocketSendMessage);
    this.webSocketService &&
        this.webSocketService.sendMessage(
          this.webSocketSendEndpoint,
          this.webSocketSendMessage
        );
  }

  clearSendWSMessage(){
    console.warn("Clearing Send WS Message content");
    this.webSocketSendMessage = "";
  }

  private sendMessage(): void {
    this.webSocketService &&
      this.webSocketService.sendMessage(
        this.webSocketSendEndpoint,
        this.webSocketInitMessage
      );
  }
}
</script>
