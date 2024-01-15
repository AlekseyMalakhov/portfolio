<template>
    <div :class="$style.container">
        <BigTransparentButton @click="saveMessage()" name="Save" :admin="admin" v-if="!message.id" :disabled="disabledOthers" />
        <BigTransparentButton @click="updateMessage()" name="Update" :admin="admin" v-else :disabled="disabledOthers" />
        <n-button type="primary" :class="$style.publish" @click="sendMessage()" :color="admin ? '#117bd4' : null" :disabled="disabledSend"
            >Send</n-button
        >
    </div>
</template>

<script>
import { useStore } from "vuex";
import { useMessage } from "naive-ui";
import BigTransparentButton from "./BigTransparentButton.vue";
import communicationsAPI from "../api/communications";
import { useRouter } from "vue-router";
import { createISODateString } from "../utilities";

export default {
    name: "CommunicationsSaveSend",
    props: {
        message: Object,
        mobile: Boolean,
        admin: Boolean,
        disabledSend: Boolean,
        disabledOthers: Boolean,
    },
    components: { BigTransparentButton },
    setup(props) {
        const store = useStore();
        const message = useMessage();
        const router = useRouter();

        const saveMessage = () => {
            const now = Date.now() + 600000;
            const sd = new Date(props.message.schedule_send_date);
            const sendDate = Date.parse(sd);
            if (sendDate && sendDate <= now) {
                message.error("Error! Scheduled send time should be at least 10 minutes later than current time.");
                return;
            }
            const mes = { ...props.message };
            if (props.disabledSend) {
                mes.schedule_send_date = null;
            }
            if (mes.schedule_send_date) {
                mes.schedule_send_date = createISODateString(mes.schedule_send_date);
            }
            store.dispatch("Manage/setLoading", true);
            communicationsAPI
                .saveMessage(mes)
                .then((response) => {
                    if (response.status === 200) {
                        message.success("Message saved successfully!");
                        store.dispatch("Manage/setEditedMessage", response.data);
                    } else {
                        message.error("Error! Please try again later!");
                    }
                    store.dispatch("Manage/setLoading", false);
                })
                .catch((err) => {
                    console.log(err);
                    message.error("Error! Please try again later!");
                    store.dispatch("Manage/setLoading", false);
                });
        };

        const updateMessage = () => {
            const now = Date.now() + 600000;
            const sd = new Date(props.message.schedule_send_date);
            const sendDate = Date.parse(sd);
            if (sendDate && sendDate <= now) {
                message.error("Error! Scheduled send time should be at least 10 minutes later than current time.");
                return;
            }
            const mes = { ...props.message };
            if (props.disabledSend) {
                mes.schedule_send_date = null;
            }
            if (mes.schedule_send_date) {
                mes.schedule_send_date = createISODateString(mes.schedule_send_date);
            }
            store.dispatch("Manage/setLoading", true);
            communicationsAPI
                .updateMessage(mes, mes.id)
                .then((response) => {
                    if (response.status === 200) {
                        message.success("Message updated successfully!");
                        store.dispatch("Manage/setEditedMessage", response.data);
                    } else {
                        message.error("Error! Please try again later!");
                    }
                    store.dispatch("Manage/setLoading", false);
                })
                .catch((err) => {
                    console.log(err);
                    message.error("Error! Please try again later!");
                    store.dispatch("Manage/setLoading", false);
                });
        };

        const sendMessage = async () => {
            store.dispatch("Manage/setLoading", true);
            let id;
            if (!props.message.id) {
                const savedMessage = await communicationsAPI.saveMessage(props.message);
                if (savedMessage.status !== 200) {
                    message.error("Error! Please try again later!");
                    return;
                }
                id = savedMessage.data.id;
            } else {
                const updatedMessage = await communicationsAPI.updateMessage(props.message, props.message.id);
                if (updatedMessage.status !== 200) {
                    message.error("Error! Please try again later!");
                    return;
                }
                id = updatedMessage.data.id;
            }
            communicationsAPI
                .sendMessage(id)
                .then((response) => {
                    if (response.status === 200) {
                        message.success("Message has been sent successfully!");
                        router.push("/user/communications");
                    } else {
                        message.error("Error! Please try again later!");
                    }
                    store.dispatch("Manage/setLoading", false);
                })
                .catch((err) => {
                    console.log(err);
                    message.error("Error! Please try again later!");
                    store.dispatch("Manage/setLoading", false);
                });
        };

        return {
            saveMessage,
            sendMessage,
            updateMessage,
        };
    },
};
</script>

<style module>
.container {
    display: flex;
    flex-direction: row;
    justify-content: center;
    width: 100%;
}
.publish {
    width: 97px;
    height: 48px;
    margin-left: 10px;
    border-radius: 8px;
    font-size: 16px;
}
.publish :global .n-button__content {
    font-weight: 700;
}
</style>
