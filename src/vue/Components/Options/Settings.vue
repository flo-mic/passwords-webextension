<template>
    <div class="settings-general">
        <translate tag="h3" say="AutofillSettings"/>
        <div class="setting">
            <slider-field id="paste-autoclose" v-model="autoclose"/>
            <translate tag="label" for="paste-autoclose" say="SettingsPastePopupClose"/>
        </div>
        <div class="setting">
            <slider-field id="paste-autosubmit" v-model="autosubmit"/>
            <translate tag="label" for="paste-autosubmit" say="SettingsPasteFormSubmit"/>
        </div>
        <div class="setting">
            <slider-field id="paste-compromised" v-model="compromised"/>
            <translate tag="label" for="paste-compromised" say="SettingsPasteWarnCompromised"/>
        </div>
        <div class="setting">
            <slider-field id="paste-autofill" v-model="autofill"/>
            <translate tag="label" for="paste-autofill" say="SettingsPasteAutofillEnabled"/>
            <help-text type="warning" text="HelpPasteAutofill"/>
        </div>
        <div class="setting">
            <slider-field id="paste-basic-auth" v-model="basicAuth"/>
            <translate tag="label" for="paste-basic-auth" say="SettingsPasteBasicAuth"/>
            <help-text type="warning" text="HelpPasteBasicAuth"/>
        </div>
        <div class="setting">
            <slider-field id="clipboard-clear-passwords" v-model="clearClipboard"/>
            <translate tag="label" for="clipboard-clear-passwords" say="SettingsClearClipboardPasswords"/>
            <help-text type="info" text="HelpClearClipboardPasswords"/>
        </div>
        <div class="setting">
            <translate tag="label" for="clipboard-clear-delay" say="SettingsClearClipboardDelay"/>
            <select-field id="clipboard-clear-delay" :options="clearClipboardDelayOptions" v-model="clearClipboardDelay" :disabled="!clearClipboard"/>
        </div>

        <translate tag="h3" say="RecommendationSettings"/>
        <div class="setting">
            <translate tag="label" for="search-recommendation-option" say="SettingsSearchRecommendationOption"/>
            <select-field id="search-recommendation-option" :options="recommendationOptions" v-model="recSearchMode"/>
        </div>
        <div class="setting">
            <translate tag="label" for="search-recommendation-maxRows" say="SettingsSearchRecommendationMaxRows"/>
            <select-field id="search-recommendation-maxRows" :options="recommendationMaxRows" v-model="recSearchRows"/>
        </div>

        <translate tag="h3" say="NotificationSettings"/>
        <div class="setting">
            <slider-field id="notification-password-new" v-model="notifyPwNew"/>
            <translate tag="label" for="notification-password-new" say="SettingsNotifyPasswordNew"/>
        </div>
        <div class="setting">
            <slider-field id="notification-password-update" v-model="notifyPwUpdate"/>
            <translate tag="label" for="notification-password-update" say="SettingsNotifyPasswordUpdate"/>
        </div>

        <translate tag="h3" say="SearchSettings"/>
        <div class="setting">
            <slider-field id="popup-related-search" v-model="relatedSearch"/>
            <translate tag="label" for="popup-related-search" say="SettingsPopupRelatedSearch"/>
        </div>
    </div>
</template>

<script>
    import Translate from '@vue/Components/Translate';
    import ErrorManager from '@js/Manager/ErrorManager';
    import SettingsService from '@js/Services/SettingsService';
    import ToastService from '@js/Services/ToastService';
    import SliderField from "@vue/Components/Form/SliderField";
    import SelectField from "@vue/Components/Form/SelectField";
    import HelpText from "@vue/Components/Options/Setting/HelpText";
    import ClipboardManager from '@js/Manager/ClipboardManager';


    export default {
        components: {HelpText, SliderField, SelectField, Translate},
        data() {
            return {
                autoclose        : false,
                autosubmit       : false,
                autofill         : false,
                basicAuth        : false,
                compromised      : false,
                notifyPwNew      : false,
                relatedSearch    : false,
                notifyPwUpdate   : false,
                recSearchMode    : 'host',
                recSearchRows    : 8,
                clearClipboard   : false,
                clearClipboardDelay: 60
            };
        },

        created() {
            this.loadData();
        },

        computed: {
            recommendationOptions() {
                return [
                    {
                        id   : 'domain',
                        label: 'LabelSearchRecommendationDomain'
                    },
                    {
                        id   : 'host',
                        label: 'LabelSearchRecommendationHost'
                    },
                    {
                        id   : 'hostport',
                        label: 'LabelSearchRecommendationHostPort'
                    },
                    {
                        id   : 'exact',
                        label: 'LabelSearchRecommendationExact'
                    }
                ];
            },
            recommendationMaxRows() {
                var i = 1;
                var result = [];
                for(i =1; i <= 20; i++) {
                    result.push({id: i, label: ['SearchRecommendationMaxRowsNumber', i]});
                }
                return result;
            },
            clearClipboardDelayOptions() {
                var i = 1;
                var result = [];
                for(let i of [15, 30, 45, 60, 90]) {
                    result.push({id: i, label: ['SettingsClipboardClearDelayOptions', i]});
                }
                return result;
            }
        },

        methods: {
            loadData() {
                this.getSetting('paste.popup.close', 'autoclose');
                this.getSetting('paste.form.submit', 'autosubmit');
                this.getSetting('paste.compromised.warning', 'compromised');
                this.getSetting('paste.autofill', 'autofill');
                this.getSetting('paste.basic-auth', 'basicAuth');
                this.getSetting('popup.related.search', 'relatedSearch');
                this.getSetting('notification.password.new', 'notifyPwNew');
                this.getSetting('notification.password.update', 'notifyPwUpdate');
                this.getSetting('search.recommendation.mode', 'recSearchMode');
                this.getSetting('search.recommendation.maxRows', 'recSearchRows');
                this.getSetting('clipboard.clear.passwords', 'clearClipboard');
                this.getSetting('clipboard.clear.delay', 'clearClipboardDelay');
            },
            async getSetting(name, variable) {
                try {
                    this[variable] = await SettingsService.getValue(name);
                } catch(e) {
                    ErrorManager.logError(e);
                    ToastService.error(e.message).catch(ErrorManager.catch);
                }
            },
            async setSetting(name, value) {
                try {
                    await SettingsService.set(name, value);
                } catch(e) {
                    ErrorManager.logError(e);
                    ToastService.error(e.message).catch(ErrorManager.catch);
                }
            }
        },

        watch: {
            autosubmit(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('paste.form.submit', value);
                }
            },
            autoclose(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('paste.popup.close', value);
                }
            },
            compromised(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('paste.compromised.warning', value);
                }
            },
            autofill(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('paste.autofill', value);
                }
            },
            basicAuth(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('paste.basic-auth', value);
                }
            },
            clearClipboard(value, oldValue) {
                if(value === true) ClipboardManager.requestReadPermission();
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('clipboard.clear.passwords', value);
                }
            },
            clearClipboardDelay(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('clipboard.clear.delay', value);
                }
            },
            relatedSearch(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('popup.related.search', value);
                }
            },
            notifyPwNew(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('notification.password.new', value);
                }
            },
            notifyPwUpdate(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('notification.password.update', value);
                }
            },
            recSearchMode(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('search.recommendation.mode', value);
                }
            },
            recSearchRows(value, oldValue) {
                if(oldValue !== null && value !== oldValue) {
                    this.setSetting('search.recommendation.maxRows', value);
                }
            }
        }
    };
</script>

<style lang="scss">
.debug-settings,
.settings-general {
    h3 {
        margin : 1.5rem 1rem .5rem;
    }
    p {
        margin : 1.5rem 1rem .5rem;
    }

    .setting {
        padding     : 0.5rem 1rem;
        display     : flex;
        align-items : center;
        gap         : .25rem;

        label {
            line-height : 2rem;
            flex-grow   : 1;
            cursor      : pointer;
        }

        .input-slider {
            flex-grow : 0;
        }

        .settings-help-text {
            margin-right : -.5rem;
        }
    }
}
</style>