<script setup>
import {Head, useForm, usePage} from "@inertiajs/vue3";
import {TabGroup, TabList, Tab, TabPanels, TabPanel} from '@headlessui/vue'
import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";
import TabItem from "@/Pages/Profile/Partials/TabItem.vue";
import {computed, ref} from "vue";
import {XMarkIcon, CameraIcon, CheckIcon} from '@heroicons/vue/24/solid'
import PrimaryButton from "@/Components/PrimaryButton.vue";
import InviteUserModal from "@/Pages/Group/InviteUserModal.vue";
import UserItem from "@/Components/app/UserItem.vue";
import TextInput from "@/Components/TextInput.vue";

const imagesForm = useForm({
    thumbnail: null,
    cover: null,
});

const showNotification = ref(true);
const coverImageSrc = ref('');
const thumbnailImageSrc = ref('');
const showInviteUserModal = ref(false);
const authUser = usePage().props.auth.user;
const searchKeyword = ref('');

const isUserAdmin = computed(() => props.group.role === 'admin');
const isJoinedToGroup = computed(() => !!props.group.role && props.group.status === 'approved');

const props = defineProps({
    errors: Object,
    users: Array,
    requests: Array,
    success: String,
    group: Object,
});

function onCoverChange(event) {
    imagesForm.cover = event.target.files[0];
    if (imagesForm.cover) {
        const reader = new FileReader();
        reader.onload = () => {
            coverImageSrc.value = reader.result;
        }
        reader.readAsDataURL(imagesForm.cover);
    }
}

function resetCoverImage() {
    imagesForm.cover = null;
    coverImageSrc.value = null;
}

function submitCoverImage() {
    imagesForm.post(route('group.updateImage', props.group.slug), {
        onSuccess: () => {
            showNotification.value = true;
            resetCoverImage();
            setTimeout(() => {
                showNotification.value = false;
            }, 3000);
        }
    });
}

function onThumbnailChange(event) {
    imagesForm.thumbnail = event.target.files[0];
    if (imagesForm.thumbnail) {
        const reader = new FileReader();
        reader.onload = () => {
            thumbnailImageSrc.value = reader.result;
        }
        reader.readAsDataURL(imagesForm.thumbnail);
    }
}

function resetThumbnailImage() {
    imagesForm.thumbnail = null;
    thumbnailImageSrc.value = null;
}

function submitThumbnailImage() {
    imagesForm.post(route('group.updateImage', props.group.slug), {
        onSuccess: () => {
            showNotification.value = true;
            resetThumbnailImage();
            setTimeout(() => {
                showNotification.value = false;
            }, 3000);
        }
    });
}

function joinToGroup() {
    const form = useForm({})
    form.post(route('group.join', props.group.slug))
}

function approveUser(user, action) {
    const form = useForm({
        user_id: user.id,
        action
    })
    form.post(route('group.approveRequest', props.group.slug))
}

</script>

<template>
    <Head title="Группа"/>

    <AuthenticatedLayout>
        <div class="container max-w-[768px] mx-auto h-full overflow-auto">
            <div class="px-4">
                <div
                    v-show="showNotification && success"
                    class="my-2 py-2 px-3 font-medium text-sm bg-emerald-500 text-white"
                >
                    {{ success }}
                </div>
                <div
                    v-if="errors.cover"
                    class="my-2 py-2 px-3 font-medium text-sm bg-red-400 text-white"
                >
                    {{ errors.cover }}
                </div>
                <div class="group relative bg-white">
                    <img class="h-[200px] object-cover w-full"
                         :src="coverImageSrc || group.cover_url"
                         alt="cover">
                    <div v-if="isUserAdmin" class="absolute top-2 right-2">
                        <button
                            v-if="!coverImageSrc"
                            class="bg-gray-50 hover:bg-gray-100 text-gray-800 py-1 px-2 text-xs flex items-center opacity-0 group-hover:opacity-100">
                            <CameraIcon class="h-3 w-3 mr-2"/>
                            Обновить обложку
                            <input type="file" class="absolute left-0 right-0 bottom-0 top-0 opacity-0"
                                   @change="onCoverChange"/>
                        </button>
                        <div v-else class="flex gap-2 bg-white p-2 opacity-0 group-hover:opacity-100">
                            <button
                                @click="resetCoverImage"
                                class="flex bg-gray-50 hover:bg-gray-100 text-gray-800 py-1 px-2 text-xs items-center">
                                <XMarkIcon class="h-3 w-3 mr-2"/>
                                Отмена
                            </button>
                            <button
                                @click="submitCoverImage"
                                class="flex bg-gray-800 hover:bg-gray-900 text-gray-100 py-1 px-2 text-xs items-center">
                                <CheckIcon class="h-3 w-3 mr-2"/>
                                Сохранить
                            </button>
                        </div>
                    </div>
                    <div class="flex">
                        <div
                            class="flex items-center justify-center relative group/thumbnail ml-[48px] -mt-[64px] w-[128px] h-[128px] rounded-full">
                            <img class="w-full h-full object-cover rounded-full"
                                 :src="thumbnailImageSrc || group.thumbnail_url"
                                 alt="thumbnail">
                            <div v-if="isUserAdmin">
                                <button
                                    v-if="!thumbnailImageSrc"
                                    class="absolute left-0 right-0 bottom-0 top-0 bg-black/50 text-gray-200 rounded-full opacity-0 flex items-center justify-center group-hover/thumbnail:opacity-100">
                                    <CameraIcon class="h-8 w-8"/>
                                    <input type="file" class="absolute left-0 right-0 bottom-0 top-0 opacity-0"
                                           @change="onThumbnailChange"/>
                                </button>
                                <div v-else class="absolute top-1 right-0 flex flex-col gap-2">
                                    <button
                                        @click="resetThumbnailImage"
                                        class="h-7 w-7 flex items-center justify-center bg-red-500/80 text-white rounded-full">
                                        <XMarkIcon class="h-5 w-5"/>
                                    </button>
                                    <button
                                        @click="submitThumbnailImage"
                                        class="h-7 w-7 flex items-center justify-center bg-emerald-500 text-white rounded-full">
                                        <CheckIcon class="h-5 w-5"/>
                                    </button>
                                </div>
                            </div>
                        </div>
                        <div class="flex flex-1 justify-between items-center p-4">
                            <h2 class="font-bold text-lg">{{ group.name }}</h2>
                            <PrimaryButton
                                v-if="!authUser"
                                :href="route('login')">
                                Войти и вступить
                            </PrimaryButton>
                            <PrimaryButton
                                v-if="isUserAdmin"
                                @click="showInviteUserModal= true">
                                Пригласить
                            </PrimaryButton>
                            <PrimaryButton
                                v-if="authUser && !group.role && group.auto_approval"
                                @click="joinToGroup">
                                Вступить
                            </PrimaryButton>
                            <PrimaryButton
                                v-if="authUser && !group.role && !group.auto_approval"
                                @click="joinToGroup">
                                Запросить
                            </PrimaryButton>
                        </div>
                    </div>
                </div>
            </div>
            <div class="p-4 pt-0">
                <TabGroup>
                    <TabList class="flex bg-white">
                        <Tab v-slot="{ selected }" as="template">
                            <TabItem text="Записи" :selected="selected"></TabItem>
                        </Tab>
                        <Tab v-if="isJoinedToGroup" v-slot="{ selected }" as="template">
                            <TabItem text="Пользователи" :selected="selected"></TabItem>
                        </Tab>
                        <Tab v-if="isUserAdmin" v-slot="{ selected }" as="template">
                            <TabItem text="Запросы" :selected="selected"></TabItem>
                        </Tab>
                        <Tab v-slot="{ selected }" as="template">
                            <TabItem text="Фото" :selected="selected"></TabItem>
                        </Tab>
                    </TabList>

                    <TabPanels class="mt-2">
                        <TabPanel class="bg-white p-3 shadow">
                            Мои записи
                        </TabPanel>
                        <TabPanel v-if="isJoinedToGroup">
                            <div class="mb-3">
                                <TextInput
                                    :model-value="searchKeyword"
                                    placeholder="Введите для поиска"
                                    class="w-full"/>
                            </div>
                            <div class="grid grid-cols-2 gap-3">
                                <UserItem
                                    class="shadow rounded-lg"
                                    v-for="user of users"
                                    :user="user"
                                    :key="user.id"/>
                            </div>
                        </TabPanel>
                        <TabPanel v-if="isUserAdmin">
                            <div v-if="requests.length" class="grid grid-cols-2 gap-3">
                                <UserItem
                                    class="shadow rounded-lg"
                                    v-for="user of requests"
                                    :user="user"
                                    :key="user.id"
                                    :for-approve="true"
                                    @approve="approveUser(user, 'approve')"
                                    @reject="approveUser(user, 'reject')"/>
                            </div>
                            <div v-else class="py-8 text-center">
                                Отсутствуют запросы на вступление
                            </div>
                        </TabPanel>
                        <TabPanel class="bg-white p-3 shadow">
                            Фото
                        </TabPanel>
                    </TabPanels>
                </TabGroup>
            </div>
        </div>
    </AuthenticatedLayout>
    <InviteUserModal v-model="showInviteUserModal"/>
</template>

<style scoped>

</style>
