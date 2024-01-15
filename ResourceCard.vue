<template>
    <div :class="$style.container">
        <div :class="$style.innerContainer" :style="mobile ? 'width: 150px' : 'width: 200px'">
            <div :class="[$style.imageContainer, mobile ? $style.mobile : null]" @click="view">
                <FileIcon color="#92929D" :size="mobile ? 50 : 75" v-if="file.category === 'document'" />
                <n-image
                    :src="file.thumbnail_url"
                    :img-props="{ class: mobile ? $style.mobileImg : $style.img }"
                    draggable="false"
                    object-fit="cover"
                    v-if="file.category === 'image' || file.category === 'video'"
                    preview-disabled
                />
            </div>
            <div :class="$style.fileName" @click="view">{{ file.name }}</div>
            <div :class="$style.date">{{ getMonthDayYear(file.create_date) }}</div>
        </div>
    </div>
</template>

<script>
import { getMonthDayYear } from "../utilities";
import FileIcon from "../icons/FileIcon.vue";
import { useRouter, useRoute } from "vue-router";
import { useStore } from "vuex";
import { computed } from "vue";
import resourceAPI from "../api/resource";
import { useMessage } from "naive-ui";

export default {
    name: "ResourceCard",
    components: { FileIcon },
    props: {
        file: Object,
        mobile: Boolean,
        cancelModal: Function,
    },
    setup(props) {
        const store = useStore();
        const message = useMessage();
        const router = useRouter();
        const route = useRoute();
        const name = computed(() => route.name);
        const selectedFilesForMessage = computed(() => store.state.Manage.selectedFilesForMessage);
        const accountColors = computed(() => store.state.Manage.accountColors);

        const view = () => {
            localStorage.setItem("selectedResource", JSON.stringify(props.file));
            store.dispatch("Manage/setSelectedResource", props.file);
            if (name.value === "ResourcesAdmin") {
                //if opened from admin page
                router.push(`/admin/view_resource`);
            } else if (name.value === "ResourcesUser") {
                //if opened from user page
                router.push(`/user/view_resource`);
            } else if (props.cancelModal) {
                //if opened from selecting for upload
                props.cancelModal();
                store.dispatch("Manage/setLoading", true);
                resourceAPI
                    .createCommunicationResourceFromOtherResource(props.file.id)
                    .then((response) => {
                        store.dispatch("Manage/setLoading", false);
                        if (response.status === 200) {
                            const list = [...selectedFilesForMessage.value];
                            list.push(response.data);
                            store.dispatch("Manage/setSelectedFilesForMessage", list);
                            message.success("File uploaded successfully!");
                        } else {
                            message.error("Error! Please try again later!");
                        }
                    })
                    .catch((err) => {
                        console.log(err);
                        message.error("Error! Please try again later!");
                        store.dispatch("Manage/setLoading", false);
                    });
            }
        };
        return {
            getMonthDayYear,
            view,
            accountColors,
        };
    },
};
</script>

<style module>
.container {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-bottom: 50px;
}
.innerContainer {
    width: 200px;
    word-break: break-word;
}
.imageContainer {
    position: relative;
    width: 200px;
    height: 210px;
    background: #f5f5f5;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    border: 1px solid #f5f5f5;
}
.mobile {
    width: 150px;
    height: 150px;
}
.imageContainer:hover {
    border: 1px solid;
    border-color: v-bind("accountColors.accent");
}
.fileName {
    margin-top: 21px;
    font-size: 16px;
    font-weight: 600;
    letter-spacing: 0.008em;
    color: #11142d;
    cursor: pointer;
    text-align: center;
}
.date {
    font-size: 12px;
    color: #92929d;
    margin-top: 3px;
    text-align: center;
}
:global .n-image :local .img {
    width: 200px;
    height: 210px;
    user-select: none;
    border-radius: 8px;
}
:global .n-image :local .mobileImg {
    width: 150px;
    height: 150px;
    user-select: none;
    border-radius: 6px;
}
</style>
