<template>
    <div>
        <button @click="openModal" class="btn btn-outline-primary">Add Channel</button>

        <!-- show list of channels -->
        <div class="mt-4">
            <button v-for="channel in channels" class="list-group-item list-group-item-action" type="button" :class="{'active': setActiveChannel(channel)}" @click="changeChannel(channel)">{{ channel.name }}</button>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="channelModal">
          <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">

              <div class="modal-header">
                <h5 class="modal-title text-dark">Modal title</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>

              <div class="modal-body">
                <form>
                    <div class="form-group">
                        <input v-model="new_channel" type="text" id="new_channel" name="new_channel" placeholder="Channel name" class="form-control">
                    </div>
                    <!-- errors -->
                    <ul class="list-group" v-if="hasErrors">
                        <li class="list-group-item text-danger" v-for="error in errors">{{ error }}</li>
                    </ul>
                </form>
              </div>

              <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancle</button>
                <button @click="addChannel" type="button" class="btn btn-primary">Add Channel</button>
              </div>
            </div>
          </div>
        </div>
    </div>
</template>

<script>
import database from 'firebase/database'
import {mapGetters} from 'vuex'

    export default {
        name: 'channels',

        data() {
            return {
                new_channel: '',
                errors: [],
                channelsRef: firebase.database().ref('channels'),
                channels: [],
                channel: null
            }
        },

        computed: {
            ...mapGetters(['currentChannel']),

            hasErrors() {
                return this.errors.length > 0
            }
        },

        methods: {
            openModal() {
                $('#channelModal').appendTo("body").modal('show');
            },

            // add channel
            addChannel() {
                this.errors = []
                // get key to the newly creating channel
                let key = this.channelsRef.push().key
                console.log('newly creating channel key: ', key)
                // minimum info needed to create a new channel
                // id and name
                let newChannel = {id: key, name: this.new_channel}
                // create new channel
                this.channelsRef.child(key).update(newChannel)
                .then(() => {
                    this.new_channel = ''
                    $("#channelModal").modal('hide')
                })
                // error handling
                .catch((error) => {
                    this.errors.push(error.message)
                })
            },

            addListeners() {
                this.channelsRef.on('child_added', snapshot => {
                    // console.log('listening channelsRef on child_added: ', snapshot.val())
                    this.channels.push(snapshot.val())

                    // set current channel
                    if(this.channels.length > 0) {
                        // set the first one as current channel
                        this.channel = this.channels[0]
                        // dispatch current channel to store
                        this.$store.dispatch("setCurrentChannel", this.channel) // pick the first one
                    }
                })
            },


            // set active channel
            setActiveChannel(channel) {
                return channel.id === this.currentChannel.id
            },

            // change channel
            changeChannel(channel) {
                this.$store.dispatch('setPrivate', false)
                this.$store.dispatch("setCurrentChannel", channel)
            },

            detachListeners() {
                this.channelsRef.off()
            }
        },

        mounted() {
            this.addListeners()
        },

        beforeDestroy() {
            this.detachListeners()
        }
    }
</script>










