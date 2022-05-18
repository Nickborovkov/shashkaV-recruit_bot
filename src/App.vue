<template>
    <b-row align-h="center" style="margin: 0; padding: 0;">
        <b-col xs="12" md="6" lg="4" class="mt-1">
            <b-card>
                    <h5 class="mb-3">Запись на консультацию</h5>

                    <b-form-group label="Ваше имя">
                        <b-form-input
                                v-model="newBotMessage.userName"
                                :formatter="stringFormatter"
                                @blur="$v.newBotMessage.userName.$touch"
                        />
                        <span class="text-danger" v-if="!$v.newBotMessage.userName.required && $v.newBotMessage.userName.$dirty">
                            Поле обязательно
                        </span>
                    </b-form-group>

                    <b-form-group label="Ваш телефон">
                        <b-form-input
                                v-model="newBotMessage.userPhone"
                                v-mask="'+7(###)###-##-##'"
                                @blur="$v.newBotMessage.userPhone.$touch"
                        />
                        <span class="text-danger" v-if="!$v.newBotMessage.userPhone.required && $v.newBotMessage.userPhone.$dirty">
                            Поле обязательно
                        </span>
                        <span class="text-danger" v-if="!$v.newBotMessage.userPhone.minLength && $v.newBotMessage.userPhone.$dirty">
                                    Введите корректный номер телефона
                                </span>
                    </b-form-group>

                    <b-form-group label="Ваш email">
                        <b-form-input
                                v-model="newBotMessage.userEmail"
                                @blur="$v.newBotMessage.userEmail.$touch"
                        />
                        <div class="text-danger" v-if="!$v.newBotMessage.userEmail.required && $v.newBotMessage.userEmail.$dirty">
                            Поле обязательно
                        </div>
                        <div class="text-danger" v-if="!$v.newBotMessage.userEmail.email && $v.newBotMessage.userEmail.$dirty">
                            Введите корректный email
                        </div>
                    </b-form-group>

                    <b-form-group label="Откуда узнали">
                        <b-form-input
                                v-model="newBotMessage.userInfocenters"
                                @blur="$v.newBotMessage.userInfocenters.$touch"
                        />
                        <span class="text-danger" v-if="!$v.newBotMessage.userInfocenters.required && $v.newBotMessage.userInfocenters.$dirty">
                            Поле обязательно
                        </span>
                    </b-form-group>

                    <b-form-group :label="isQa ? 'Тема обращения 👎 👎 👎' : 'Тема обращения'">
                        <b-form-textarea
                                v-model="newBotMessage.userTheme"
                                @blur="$v.newBotMessage.userTheme.$touch"
                        />
                        <span class="text-danger" v-if="!$v.newBotMessage.userTheme.required && $v.newBotMessage.userTheme.$dirty">
                            Поле обязательно
                        </span>
                    </b-form-group>

                    <b-button variant="info" block @click="sendBotMessage">Отправить</b-button>
                    <b-alert class="mt-2" style="text-align: center" variant="danger" v-model="error">Ошибка отправки</b-alert>
            </b-card>
        </b-col>
    </b-row>
</template>

<script>
    import { required, email, minLength } from 'vuelidate/lib/validators'
    import {instance} from "./api/instance";

    export default {
        name: 'NewBotMessage',
        data() {
            return {
                newBotMessage: {
                    userName: '',
                    userPhone: '',
                    userEmail: '',
                    userInfocenters: '',
                    userTheme: ''
                },
                fields: [],
                message: '',
                isQa: false,
                error: false
            }
        },
        validations: {
            newBotMessage: {
                userName: { required },
                userPhone: { required, minLength: minLength(16) },
                userEmail: { required, email },
                userInfocenters: { required },
                userTheme: { required }
            },
        },
        watch: {
            newBotMessage: {
                deep: true,
                handler: function () {
                    if(this.newBotMessage.userTheme.toLowerCase().includes('qa')
                    || this.newBotMessage.userTheme.toLowerCase().includes('qa-engineer')
                    || this.newBotMessage.userTheme.toLowerCase().includes('тестер')
                    || this.newBotMessage.userTheme.toLowerCase().includes('тестировщик')){
                        this.isQa = true
                    }else {
                        this.isQa = false
                    }
                }
            },
            error() {
                if(this.error) {
                    setTimeout(() => {this.error = false}, 2000)
                }
            }
        },
        methods: {
            async sendBotMessage() {
                this.$v.$touch()
                if(this.$v.$dirty && this.$v.$anyError){
                    window.scrollTo(0 ,0)
                    return
                }

                this.error = false
                this.fields = []
                this.message = ''


                this.fields = [
                    '<b>Имя</b>: ' + this.newBotMessage.userName,
                    '<b>Телефон</b>: ' + this.newBotMessage.userPhone,
                    '<b>Email</b>: ' + this.newBotMessage.userEmail,
                    '<b>Откуда узнали</b>: ' + this.newBotMessage.userInfocenters,
                    '<b>Тема сообщения</b>: ' + this.newBotMessage.userTheme,
                ]

                this.fields.forEach(field => {
                    this.message += field + '\n'
                })

                try {
                    const response = await instance.post(`/sendMessage?chat_id=-452132239&parse_mode=html&text=${encodeURI(this.message)}`)
                    if(response.status === 200){
                        this.clearFields()
                        this.$v.$reset()
                    }else {
                        this.error = true
                    }
                }catch (e) {
                    this.error = true
                }

            },
            stringFormatter(value) {
                return value.replace(/[^a-zа-яё]/gi, '')

            },
            clearFields() {
                for(const key of Object.keys(this.newBotMessage)){
                    this.newBotMessage[key] = ''
                }
            }
        }
    }
</script>

