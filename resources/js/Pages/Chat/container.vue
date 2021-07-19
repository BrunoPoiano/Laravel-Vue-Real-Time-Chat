<template>
    <app-layout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                <chat-room-selection
                    v-if="currteRoom.id"
                    :rooms="chatRooms"
                    :currteRoom="currteRoom"
                    v-on:roomchanged="setRoom($event)"
                />
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                    <message-container :messages="messages" />
                    <input-message
                        :room="currteRoom"
                        v-on:messageSent="getMessages()"
                    />
                </div>
            </div>
        </div>
    </app-layout>
</template>

<script>
import AppLayout from "@/Layouts/AppLayout";
import MessageContainer from "./messageContainer.vue";
import InputMessage from "./inputMessage.vue";
import ChatRoomSelection from "./chatRoomSelection.vue";

export default {
    components: {
        AppLayout,
        MessageContainer,
        InputMessage,
        ChatRoomSelection,
    },
    data: function () {
        return {
            chatRooms: [],
            currteRoom: [],
            messages: [],
        };
    },
    watch:{
        currteRoom(val, oldVal){
            if(oldVal.id){
                this.disconect(oldVal)
            }
            this.connect();
        }   
    },
    methods: {
        connect() {
            if (this.currteRoom.id) {
                let vm = this;
                this.getMessages();
                window.Echo.private("chat." + this.currteRoom.id)
                .listen('.message.new', e =>{
                    vm.getMessages();
                })
            }
        },
        disconect( room ){
            window.Echo.leave("chat." +room.id);
        },
        getRooms() {
            axios
                .get("/chat/rooms")
                .then((response) => {
                    this.chatRooms = response.data;
                    this.setRoom(response.data[0]);
                })
                .catch((error) => {
                    console.log(error);
                });
        },
        setRoom(room) {
            this.currteRoom = room;
            
        },
        getMessages() {
            console.log("aqui");
            axios
                .get("/chat/room/" + this.currteRoom.id + "/messages")
                .then((response) => {
                    this.messages = response.data;
                })
                .catch((error) => {
                    console.log(error);
                });
        },
    },
    created() {
        this.getRooms();
    },
};
</script>
