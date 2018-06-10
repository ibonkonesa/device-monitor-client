<template>
    <q-page>
        <q-list highlight v-if="devices.length > 0">
            <q-list-header>Registered devices
                <small style="color:deepskyblue">click device to see details</small>
            </q-list-header>
            <q-item @click.native="detail = device" :key="device.mac" v-for="device in devices">
                <q-item-main :label="device.ip"/>
                <q-item-side right>{{device.desc}}</q-item-side>
            </q-item>
        </q-list>
        <div v-else>
            <h4 style="text-align: center">There are no devices registered yet</h4>
        </div>
        <div style="padding: 0 1rem 2rem 1rem" v-if="detail !== null">
            <h4>IP:
                <small>{{detail.ip}}</small>
            </h4>
            <h4>MAC:
                <small>{{detail.mac}}</small>
            </h4>
            <h4>DESC:
                <small>{{detail.desc}} <font @click="setNewName" style="color: deepskyblue">change</font></small>
            </h4>
            <h4>LAST UPDATE:
                <small>{{detail.timestamp | date}}</small>
            </h4>
            <q-btn @click="addKeep" v-if="!detail.keep" color="primary" label="Keep device"/>
            <q-btn @click="disableKeep" v-else color="secondary" label="Disable keeping"/>
        </div>
        <div v-else style=" padding-top: 2em; text-align: center">


            <h4>
                Tap a device to see details
            </h4>

        </div>
    </q-page>
</template>

<style>
</style>

<script>

    import Firebase from 'firebase'
    import {config} from '../config/env';

    let app = Firebase.initializeApp(config);
    let db = app.database();
    let devicesRef = db.ref('devices');

    export default {
        name: 'PageIndex',
        firebase: {
            devices: devicesRef
        },
        filters: {
            date: function (value) {
                let date = new Date(value * 1000);
                return date.toLocaleDateString() + ' ' + date.toLocaleTimeString();
            },
        },
        data() {
            return {
                detail: null,
                messages: [
                    {topic: 'deviceNew', message: 'Device has joined'},
                    {topic: 'deviceDelete', message: 'Device has left'},
                ],
                newDesc: null,
                isLoaded: false,
            }
        },

        methods: {

            setNewName: function () {
                this.newDesc = this.detail.desc;
                this.$q.dialog({
                    title: 'Change description',
                    message: 'You can set a name to this device',
                    prompt: {
                        model: this.newDesc,
                        type: 'text'
                    }
                }).then((desc) => {
                    this.$firebaseRefs.devices.child(this.detail['.key']).update({desc: desc}).then(x => {
                        this.newDesc = null;
                        this.detail.desc = desc
                    })
                })
            },

            addKeep: function () {
                this.$firebaseRefs.devices.child(this.detail['.key']).update({keep: true}).then(this.detail.keep = true)
            },

            disableKeep: function () {
                this.$firebaseRefs.devices.child(this.detail['.key']).update({keep: false}).then(this.detail.keep = false)
            }

        },
        created() {

            this.$watch('devices', function (newData) {
                if (!this.isLoaded) {
                    //WAIT FIREBASE DATA IS LOADED TO HIDE SPLASH SCREEN
                    navigator.splashscreen.hide();
                    this.isLoaded = true;
                } else if (this.detail !== null) {
                    //UPDATE DETAIL ITEM
                    this.detail = newData.find(item => item.mac === this.detail.mac);
                }
            });
            //REGISTER TO PUSH TOPICS
            FCMPlugin.subscribeToTopic('deviceNew');
            FCMPlugin.subscribeToTopic('deviceDelete');
            //DEAL MESSAGE RECEPTION INTO THE APP
            FCMPlugin.onNotification((data) => {
                if (!data.wasTapped) {
                    this.$q.notify(
                        {
                            message: this.messages.find(item => item.topic === data.topic).message,
                            position: 'bottom'

                        }
                    )
                }
            });
        }
    }
</script>
