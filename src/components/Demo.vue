<template>
    <form @submit.prevent="voiceText">
        <fieldset class="uk-fieldset">

            <legend class="uk-legend">{{ $t('demo.try') }}</legend>

            <div class="uk-margin">
                <div class="uk-form-label">{{ $t('demo.voice') }}<sup style="color: darkred; font-size: 10pt">*</sup></div>
                <div class="uk-form-controls">
                    <select v-model="chosen_voice" class="uk-select">
                        <option v-for="(v) in voices" 
                          :key="v.public_id" :value="v">{{ v.name }}</option>
                    </select>
                </div>
            </div>

            <div class="uk-margin">
                <div class="uk-form-label">{{ $t('demo.text') }}<sup style="color: darkred; font-size: 10pt">*</sup></div>
                <textarea v-model="text" class="uk-textarea" rows="5" :placeholder="$t('demo.holder')"></textarea>
            </div>

            <div class="uk-margin">
                <button class="uk-button uk-button-default">
                    {{ $t('demo.submit') }}
                </button>
            </div>

            <center class="uk-margin" v-if="state == 1">
                <span uk-spinner="ratio: 3.9"></span>
            </center>
            <center class="uk-margin uk-align-center" v-if="state == 2">
                <audio controls>
                    <source :src="result_link" type="audio/wav">
                </audio>
            </center>

        </fieldset>
    </form>
</template>

<script>
import * as Tone from 'tone'

export default {
    name: 'DemoVoiceOverForm',
    data: function () {
        return {
            state: 0,
            result_link: "",
            chosen_voice: null,
            text: "",
            voices: null,
            player: null
        }
    },
    mounted: function () {
        var _this = this;
        this.$http
            .get('http://34.118.9.73:8080/api/voice/general')
            .then(resp => {
                _this.voices = resp.data
            })
    },
    methods: {
        voiceText: async function () {
            await Tone.start();
            console.log('audio is ready');

            var _this = this;
            if (_this.text == "" || _this.chosen_voice == null) {
                UIkit.notification({
                    message: "Fill in the fields and choose the voice",
                    status: 'danger',
                    pos: 'top-right',
                    timeout: 5000
                });
                return;
            }
            _this.state = 1;
            this.$http
                .post('http://34.118.9.73:8080/api/query/demo', {
                    public_id: _this.chosen_voice.public_id,
                    text: _this.text
                })
                .then(resp => {
                    _this.result_link = resp.data.result;
                    _this.state = 2;
                })
        }
    }
}
</script>
