<template>
    <q-page>

        <q-list highlight v-if="devices.length > 0">
            <q-list-header>Registered devices</q-list-header>
            <q-item @click.native="detail = device" :key="device.mac" v-for="device in devices">
                <q-item-main :label="device.ip"/>
                <q-item-side right>{{device.desc}}</q-item-side>
            </q-item>
        </q-list>

        <div v-else>

            <h4 style="text-align: center">There are no devices registered yet</h4>

        </div>

        <div style="padding: 0 1rem 0 1rem" v-if="detail !== null">
            <h4>IP:
                <small>{{detail.ip}}</small>
            </h4>

            <h4>MAC:
                <small>{{detail.mac}}</small>
            </h4>

            <h4>DESC:
                <small>{{detail.desc}}</small>
            </h4>

            <h4>LAST UPDATE:
                <small>{{detail.timestamp | date}}</small>
            </h4>
        </div>
    </q-page>
</template>

<style>
</style>

<script>

    import Firebase from 'firebase'
    import {config} from '../config/env';

    //UPDATE THIS INFO

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
                console.log(value);
                let date = new Date(value*1000);
                return date.toLocaleDateString()+' '+date.toLocaleTimeString();
            },
        },
        data() {
            return {
                detail: null,
                messages: [
                    {topic: 'deviceNew', message: 'Device has joined'},
                    {topic: 'deviceDelete', message: 'Device has left'},
                ]
            }
        },
        created() {
            //REGISTER TO PUSH TOPICS
            FCMPlugin.subscribeToTopic('deviceNew');
            FCMPlugin.subscribeToTopic('deviceDelete');
            //DEAL MESSAGE RECEPTION INTO THE APP
            FCMPlugin.onNotification((data) => {
                if (!data.wasTapped) {
                    this.$q.notify(this.messages.find(item => item.topic === data.topic).message)
                }
            });

            //CLOSE SPLASHSCREEN!
            navigator.splashscreen.hide();

        }
    }
</script>
