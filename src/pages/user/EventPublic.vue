<template>
    <q-page class="q-pb-xl bg-roxo-light animate__animated animate__fadeInLeft ">
        <div class="w100 q-pt-sm q-pl-sm" >
            <q-btn @click="returnBack()" icon="keyboard_return" color="secondary" glossy></q-btn>
        </div>
        <div class="w100 row justify-center q-mt-md q-px-md">
            <q-card v-if="eventoIndisponivel">
                <q-card-section>
                    <div id="title-2" class="text-center  text-secondary text-bold q-py-md">Evento Indisponível<br>( ˘︹˘ )</div>
                </q-card-section>
            </q-card>
        </div>
        <div v-if="!loading && !eventoIndisponivel" class="animate__animated animate__fadeInRight w100 text-primary text-center q-px-sm " id="title">
            {{ event.title }}
        </div>
        <q-btn  v-if="!loading && !eventoIndisponivel" :to="'/' + event.host_login" class="text-secondary text-bold w100 text-center q-mb-md text-" flat :label="event.host"></q-btn>
        <div v-if="isMobile && !loading && !eventoIndisponivel" class="w100 q-mb-lg row justify-center">
            <a class="text-white bg-green-14 q-pa-md rounded-borders text-bold shadow-1" style="text-decoration: none;" href="#ingressos">Comprar Ingressos <q-icon size="sm" name="add_shopping_cart"></q-icon></a>
        </div>
        <div v-if="!loading && !eventoIndisponivel" id="cards-wrapper" class="w100 row items-start q-gutter-y-md">
            <q-card class="card-event q-mx-md bg-grey-3 q-mt-md">
                <q-card-section>
                    <q-item >
                        <q-item-section class="text-black">
                            <q-item-label id="title-2"  class="text-primary">INFORMAÇÕES</q-item-label>
                            <q-item-label class="text-bold text-grey-14 q-py-sm">{{ event.desc }}</q-item-label>
                            <q-item-label class="text-bold text-primary w100 column text-bold" >📆 {{ event.date.replaceAll('-', '/') }} ⏱️ {{ event.initial_time ? event.initial_time : 'xx:xx' }}{{ event.final_time ? (' - ' + event.final_time) : ''}}</q-item-label>
                            <img id="img-events" :src="event.img_url" class="q-mt-md" alt="🎇 Banner do Evento"/>
                            <div id="ingressos"></div>
                        </q-item-section>
                    </q-item>
                </q-card-section>
            </q-card>

            <q-card class="card-event q-mx-md bg-grey-3 q-mt-md">
                <q-card-section>
                    <q-item >
                        <q-item-section class="text-black">
                            <q-item-label id="title-2"  class="text-primary" >INGRESSOS</q-item-label>
                            <div id="ticket-types" >
                                <q-item id="ticket" v-for="(ticket, index) in event.ticket_types" :key="index" style="border-left: 6px solid #9573f3;" class="shadow-1 q-mt-md">
                                    <q-item-section class="text-bold text-primary q-py-sm" id="title-layout">
                                        <q-icon name="confirmation_number"></q-icon>{{ ticket.title }}<br><strong class="text-secondary q-pt-xs">R$ {{ formatStringValue(ticket.price) }}</strong>
                                        <q-btn @click="openModalBuyTicket(ticket)" class="q-mt-sm q-py-lg" icon="add_shopping_cart" label="Comprar" color="green-14" glossy></q-btn>
                                    </q-item-section>
                                </q-item>
                            </div>
                        </q-item-section>
                    </q-item>
                </q-card-section>
            </q-card>
            <q-card class="card-event q-mx-md bg-grey-3 q-mt-md">
                <q-card-section>
                    <q-item >
                        <q-item-section class="text-black">
                            <q-item-label id="title-2"  class="q-pt-sm text-primary">CONTATO</q-item-label>
                            <q-item-label class="text-bold text-grey-14 q-py-sm">{{ event.contact }}<br><br><strong class="text-secondary">Hospedado por {{ event.host }}</strong></q-item-label>
                            <q-item-label id="title-2"  class="text-primary q-pt-md">LOCALIZAÇÃO</q-item-label>
                            <q-item-label class="text-bold text-grey-14 q-py-sm">{{ event.address }}</q-item-label>
                            <iframe v-if="event.maps_loc" :src="event.maps_loc" class="q-mt-md w100" height="450" style="border:0;" loading="lazy"
                            referrerpolicy="no-referrer-when-downgrade"></iframe>
                        </q-item-section>
                    </q-item>
                </q-card-section>
            </q-card>
        </div>
        <q-dialog  v-model="modalBuyTicket" persistent>
            <div class="q-px-md q-pb-md bg-grey-4">
                <div class="w100 q-mt-md text-white bg-grad-2 q-py-md text-center" id="title-layout">COMPRAR INGRESSO</div>
                <div class="text-center q-pt-lg text-black text-h6 q-px-xs">Deseja realmente comprar o ingresso<br><strong class="text-primary">{{ ingressoHandle.title }}</strong><br>por<strong class="text-primary"> R$ {{ Utils.formatCurrency(ingressoHandle.totalValue, 'brl') }}</strong> ?</div>
                <div class="q-pt-md mid-opacity text-center text-bold text-primary">{{ stringTaxes() }}</div>
                <div id="person-ticket-info" class="q-mt-md q-pb-md">
                    <q-toggle color="primary" v-model="buyTicketHandler.toMe" :label="!buyTicketHandler.toMe ? 'Comprar Pra Outro' : 'Comprar Pra Mim'" @update:model-value="buyToMe()" class="q-mt-xs q-mb-md text-bold text-secondary" />
                    <q-input v-model="buyTicketHandler.ticket_person_name" label="Nome Completo*" outlined dense>
                        <template v-slot:prepend>
                            <q-icon name="person" :color="isCampoValid('name', buyTicketHandler.ticket_person_name) ? 'primary' : 'grey-8'" />
                        </template>
                    </q-input>
                    <q-input class="q-mt-sm" v-model="buyTicketHandler.ticket_person_cpf" @update:model-value="updateCpf()" mask="###.###.###-##" maxlength="14" label="CPF do Responsável*" 
                        outlined reverse-fill-mask dense>
                        <template v-slot:prepend>
                            <q-icon name="badge"  :color="isCampoValid('cpf', buyTicketHandler.ticket_person_cpf) ? 'primary' : 'grey-8'" />
                        </template>
                    </q-input>
                    <q-input class="q-mt-sm" v-model="buyTicketHandler.ticket_person_email" label="E-mail*" type="email"  outlined dense>
                        <template v-slot:prepend>
                            <q-icon name="email"  :color="isCampoValid('email', buyTicketHandler.ticket_person_email) ? 'primary' : 'grey-8'" />
                        </template>
                    </q-input>
                    <q-input class="q-mt-sm" v-model="buyTicketHandler.ticket_person_phone" mask="(##) #####-####" maxlength="15" label="Telefone*" outlined dense>
                        <template v-slot:prepend>
                            <q-icon name="phone"  :color="isCampoValid('phone', buyTicketHandler.ticket_person_phone) ? 'primary' : 'grey-8'" />
                        </template>
                    </q-input>
                    <!-- <q-input class="q-mt-sm" v-model="buyTicketHandler.ticket_person_birthday" mask="##/##/####" maxlength="10" label="Data de Nascimento" outlined  dense>
                        <template v-slot:prepend>
                            <q-icon name="cake" :color="isCampoValid('birthday', buyTicketHandler.ticket_person_birthday) ? 'primary' : 'grey-8'" />
                        </template>
                    </q-input> -->
                </div>
                <TicketPaymentComponent v-if="isCampoValid('checkMPbtn', 'checkMPbtn')" />
                <div v-else>
                    <q-btn color="primary" disabled icon-right="paid" label="Realizar Pagamento" class="w100 q-mb-md q-py-md"></q-btn>
                </div>
                <div class="w100 row justify-center">
                    <q-btn label="voltar" flat color="secondary" @click="modalBuyTicket = false" />
                </div>
            </div>
        </q-dialog>
        <div v-if="loading" class="row w100 q-pb-xl justify-center">
            <q-spinner-ball color="secondary" size="lg" />
            <q-spinner-ball color="secondary" size="lg" />
            <q-spinner-ball color="secondary" size="lg" />
        </div>
    </q-page>
</template>

<script setup>
import { onBeforeMount, ref } from "vue";
import { useRoute, useRouter } from "vue-router";
import TicketPaymentComponent from 'src/components/TicketPaymentComponent.vue'

import { api } from 'src/boot/axios';
import { SessionStorage, useQuasar } from "quasar";
import { Utils } from "src/utils/Utils";
const ingressoHandle = ref()
const isMobile = window.innerWidth < 800;
const route = useRoute();
const router = useRouter();
const event = ref(null);
const loading = ref(false);
const modalBuyTicket = ref(false);
const $q = useQuasar();
const eventoIndisponivel = ref(false);
const ticketPaymentButton = ref(false);

const buyTicketHandler = ref({
    toMe: false,
    ticket_person_name: '',
    ticket_person_email: '',
    ticket_person_cpf: '',
    ticket_person_phone: '',
    // ticket_person_birthday: '',
});

const userSession = sessionStorage.getItem('user') ? JSON.parse(sessionStorage.getItem('user')) : null
function buyToMe(){
    if(userSession == null) return
    buyTicketHandler.value.ticket_person_cpf = '';
    if(buyTicketHandler.value.toMe){
        buyTicketHandler.value.ticket_person_name = userSession.name;
        buyTicketHandler.value.ticket_person_email = userSession.email;
        buyTicketHandler.value.ticket_person_cpf = userSession.cpf;
        buyTicketHandler.value.ticket_person_phone = userSession.phone;
        // buyTicketHandler.value.ticket_person_birthday = userSession.birthday;
    } else {
        buyTicketHandler.value.ticket_person_name = '';
        buyTicketHandler.value.ticket_person_email = '';
        buyTicketHandler.value.ticket_person_phone = '';
        // buyTicketHandler.value.ticket_person_birthday = '';
    }
}

onBeforeMount(async () => {
    loading.value = true;
    
    await api.get('/events?event=' + route.params.event).then((res) => {
        event.value = res.data;
    })
    .catch((err) => {
        eventoIndisponivel.value = true;
    })
    .finally(() => {
        loading.value = false;
    });
});

function returnBack() {
    if (window.location.href.includes('ingressos')) {
        window.history.back();
        window.history.back();
    } else
    window.history.back();
}

function formatStringValue(str){
    if(typeof str === 'string') {
        str = str.includes(',') ? str.replace(',', '.') : str;
        str = Number(str);
    }
    return str.toFixed(2).toString().replace('.', ',');
}

function stringTaxes() {
    let value = ingressoHandle.value.price
    return 'R$ ' + value.toFixed(2).toString().replace('.', ',')+ ' + ' + 'R$ ' + (value * 0.08).toFixed(2).toString().replace('.', ',') + ' (8% de taxa)';
}

function openModalBuyTicket(ticket) {
    if(sessionStorage.getItem('user')){
        if(typeof ticket.price === 'string') {
            ticket.price = ticket.price.includes(',') ? ticket.price.replace(',', '.') : ticket.price;
            ticket.price = Number(ticket.price);
        }
        const ticketConfigs = {
            event_id: event.value.id,
            title: ticket.title,
            price: ticket.price,
            taxes: 0.08, // taxa de 5% pela venda do ingresso
            totalValue: ticket.price + (ticket.price * 0.08),
        }
        ingressoHandle.value = ticketConfigs;
        sessionStorage.setItem('ticketConfigs', JSON.stringify(ticketConfigs));
        modalBuyTicket.value = true;
    } else {
        $q.notify({
            message: 'Você precisa estar logado para comprar ingressos.',
            color: 'secondary',
            position: 'top',
            icon: 'local_activity',
            timeout: 4000
        });
        sessionStorage.setItem('comeFromTicketIntention', event.value.id);
        router.push('/login')
    }
}

async function updateCpf() {
    if(buyTicketHandler.value.ticket_person_cpf.length === 14) {
        await Utils.validaCPF(buyTicketHandler.value.ticket_person_cpf)
        .then((res) => {
            $q.notify({
                message: 'CPF Válido',
                color: 'green',
                position: 'top',
                icon: 'badge',
                timeout: 4000
            });
        })
        .catch((err) => {
            $q.notify({
                message: err.message,
                color: 'secondary',
                position: 'top',
                icon: 'badge',
                timeout: 4000
            });
            ok.value.cpf = false;
        });
    }
}

const ok = ref({
    name: false,
    email: false,
    cpf: false,
    phone: false,
    // birthday: false,
})

function isCampoValid(campo, valor) {
    switch (campo) {
        case 'name':
            if(valor.length < 3) {
                ok.value.name = false;
                return false;
            }
            ok.value.name = true;
            return true;
        case 'email':
            if(valor.length < 3) {
                ok.value.email = false;
                return false;
            } else if(!valor.includes('@') || !valor.includes('.')) {
                ok.value.email = false;
                return false;
            }
            ok.value.email = true;
            return true;
        case 'cpf':
            if(valor.length < 14) {
                ok.value.cpf = false;
                return false;
            }
            ok.value.cpf = true;
            return true;
        case 'phone':
            if(valor.length < 15) {
                ok.value.phone = false;
                return false;
            }
            ok.value.phone = true;
            return true;
        // case 'birthday':
        //     if(valor.length < 10) {
        //         ok.value.birthday = false;
        //         return false;
        //     }
        //     ok.value.birthday = true;
        //     return true;
        case 'checkMPbtn':
            if(ok.value.name && ok.value.email && ok.value.cpf && ok.value.phone) {
            // if(ok.value.name && ok.value.email && ok.value.cpf && ok.value.phone && ok.value.birthday) {
            sessionStorage.setItem('ticketPerson', JSON.stringify(buyTicketHandler.value));
                return true;
            }
            return false;
    }
}

</script>

<style scoped>
#img-events {
    width: 100%;
    object-fit: cover;
    border-radius: 12px;
    border-bottom: 4px solid #9573f3;
    border-right: 4px solid #9573f3;
}

.q-card {
    width: 100%;
    border-radius: 4px;
    border-left: 6px solid #6310E1;
}

#ticket{
    border-radius: 8px;
    background-color: #f9f8ff;
    cursor: pointer;
    transition: all 0.3s linear;
}

#ticket:hover {
    background-color: #e5efff;
}

@media (min-width: 800px) {
    #img-events {
        width: 100%;
    }
    #cards-wrapper{
        justify-content: center;
    }
    .q-card {
        width: 45%;
    }
}
@media (min-width: 1100px) {
    #img-events {
        width: 100%;
    }
    .q-card {
        width: 30%;
    }
    #cards-wrapper{
        flex-wrap: nowrap;
    }
}

</style>