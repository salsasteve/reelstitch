<template>
    <div>
      <!-- Display a login form if the user is not authenticated -->
      <template v-if="!user.is">
        <form @submit.prevent="login">
          <label>Username:</label>
          <input v-model="username" type="text" />
          <br />
          <label>Password:</label>
          <input v-model="password" type="password" />
          <br />
          <button type="submit">Login</button>
        </form>
      </template>
  
      <!-- Display the chat interface if the user is authenticated -->
      <template v-else>
        <!-- Display the chat messages -->
        <div v-for="message in messages" :key="message.timestamp">
          {{ message.text }}
        </div>
  
        <!-- Display the input field for sending new messages -->
        <form @submit.prevent="sendMessage">
          <input v-model="newMessage" type="text" />
          <button type="submit">Send</button>
        </form>
      </template>
    </div>
  </template>
  
  <script>
  import Gun from "gun/gun";
  
  export default {
    data() {
      return {
        // Initialize the Gun instance
        gun: new Gun(),
        // Data for the login form
        username: "",
        password: "",
        // Data for the chat interface
        user: {}, // The authenticated user
        messages: [], // An array of chat messages
        newMessage: "", // The message to be sent
      };
    },
    mounted() {
      // Check if the user is already authenticated
      this.gun.auth(function (ack) {
        if (ack.err) {
          // If the user is not authenticated, set the user data to an empty object
          this.user = {};
        } else {
          // If the user is authenticated, set the user data to the authenticated user
          this.user = ack;
          // Fetch the chat messages from the Gun database
          this.gun
            .get("messages")
            .map()
            .on((message, id) => {
              this.messages.push(message);
            });
        }
      });
    },
    methods: {
      // Method for handling the login form submission
      login() {
        // Use the gun.user().auth() method to authenticate the user
        this.gun.auth(this.username, this.password, (ack) => {
          if (ack.err) {
            // If the authentication fails, display an error message
            console.error(ack.err);
          } else {
            // If the authentication succeeds, set the user data to the authenticated user and fetch the chat messages
            this.user = ack;
            this.gun
              .get("messages")
              .map()
              .on((message, id) => {
                this.messages.push(message);
              });
          }
        });
      },
      // Method for sending a new chat message
      sendMessage() {
        // Use the gun.get().put() method to add the new message to the 'messages' path in the Gun database
        this.gun.get("messages").set({
          text: this.newMessage,
          timestamp: new Date().getTime(),
          // Set the 'by' property to the authenticated user's username
          by: this.user.alias,
        });
        // Clear the input field
        this.newMessage = "";
      },
    },
  };
  </script>
  