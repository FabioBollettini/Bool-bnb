<template>
    <div class="padding-header-fixed">
        <div class="container d-flex">
            <div class="my-container py-4 mr-5" v-if="apartment">
                <!-- Back button -->
                <router-link
                    v-if="this.$route.params.currentRoute == 'home'"
                    :to="{
                        name: 'home'
                    }"
                    class="arrow-container"
                >
                    <i class="fas fa-arrow-left back-arrow"></i>
                </router-link>

                <router-link
                    v-else
                    :to="{
                        name: 'advancedsearch'
                    }"
                    class="arrow-container"
                >
                    <i class="fas fa-arrow-left back-arrow"></i>
                </router-link>
                <!-- Title -->
                <h2 class="my-3 title">
                    <i class="fas fa-house-user"></i> {{ apartment.title }}
                </h2>
                <!-- Address -->
                <div class="mb-3">
                    <i class="fas fa-map-marker-alt mr-3 user-icon"></i
                    >{{ apartment.address }}
                </div>
                <!-- Host d kitemmurt -->
                <div class="mb-3">
                    <i class="far fa-user-circle mr-2 user-icon"></i
                    >{{ apartment.user.name }}
                </div>
                <!-- Img -->
                <div class="img-wrapper">
                    <img :src="apartment.img_path" :alt="apartment.title" />
                </div>
                <div class="d-flex my-3">
                    <!-- Rooms -->
                    <div class="mr-3">
                        <strong>Rooms:</strong> {{ apartment.rooms }}
                    </div>
                    <!-- Beds -->
                    <div class="mr-3">
                        <strong><i class="fas fa-bed"></i></strong>
                        {{ apartment.beds }}
                    </div>
                    <!-- Floor -->
                    <div class="mr-3">
                        <strong>Floor:</strong> {{ apartment.floor }}
                    </div>
                    <!-- Bathrooms -->
                    <div class="mr-3">
                        <strong><i class="fas fa-toilet"></i></strong>
                        {{ apartment.bathrooms }}
                    </div>
                    <!-- Mq -->
                    <div v-if="apartment.square_meters">
                        <strong>Mq:</strong> {{ apartment.square_meters }} mq
                    </div>
                </div>
                <!-- Services -->
                <div class="d-flex flex-wrap">
                    <div
                        v-for="(service, index) in apartment.services"
                        :key="index"
                        class="d-flex badge badge-boolbnb p-2 m-2"
                    >
                        {{ service.name }}
                    </div>
                </div>
                <!-- Description -->
                <div class="mb-3 mt-3 word-wrap">
                    <strong>Description:</strong> {{ apartment.description }}
                </div>
                <!-- Send Message -->
                <div class="d-flex justify-content-center">
                    <input
                        v-show="this.$route.name == 'apartment-details'"
                        @click="clickMessage"
                        class="mt-4 btn btn-boolbnb"
                        type="button"
                        value="Contact Host"
                    />
                </div>
            </div>
            <Maps
                class="my-3"
                v-show="this.$route.name == 'apartment-details'"
            />
        </div>
        <div v-show="clickMessageStatus" class="searchFilter pt-5">
            <div class="box-search py-2">
                <Message class="p-5" :apartment="apartment" />
                <span class="close" @click="clickMessage">Close</span>
            </div>
        </div>
    </div>
</template>

<script>
import Message from "../components/Message.vue";
import Maps from "../components/Maps.vue";
import tt from "@tomtom-international/web-sdk-maps";

export default {
    name: "Details",
    components: {
        Message,
        Maps
    },
    data() {
        return {
            apartment: null,
            clickMessageStatus: false,
            marker: "",
            map: "",
            service: tt.setProductInfo("bool", "version2")
        };
    },
    created() {
        this.getApartmentDetails();
    },
    methods: {
        getApartmentDetails() {
            axios
                .get(
                    `http://127.0.0.1:8000/api/apartments/${this.$route.params.slug}`
                )
                .then(res => {
                    this.apartment = res.data;
                    this.map = tt.map({
                        key: "gHNQlC91c7IVQOAYndiQCxDEAX09ZzVj",
                        container: "map",

                        center: [res.data.longitude, res.data.latitude],
                        zoom: 15
                    });
                    this.marker = new tt.Marker()
                        .setLngLat([res.data.longitude, res.data.latitude])
                        .addTo(this.map);
                    var popupOffsets = {
                        top: [0, 0],
                        bottom: [0, -50],
                        "bottom-right": [0, -70],
                        "bottom-left": [0, -70],
                        left: [25, -35],
                        right: [-25, -35]
                    };

                    var popup = new tt.Popup({ offset: popupOffsets }).setHTML(
                        res.data.title + "<br>" + res.data.address
                    );
                    this.marker.setPopup(popup).togglePopup();
                })
                .catch(err => {
                    console.log(err);
                });
        },

        // Open/Close the messages menu
        clickMessage() {
            this.clickMessageStatus = !this.clickMessageStatus;
        }
    }
};
</script>

<style scoped lang="scss">
.padding-header-fixed {
    padding-top: 80px;
    min-height: 100vh;
    position: relative;
}

.my-container {
    position: relative;
}

.title {
    font-weight: 600;
}

.img-wrapper {
    width: 500px;
    img {
        width: 100%;
        border-radius: 10px;
    }
}

.searchFilter {
    display: flex;
    position: absolute;
    z-index: 1031;
    background: rgba(#000000, 0.3);
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    justify-content: center;
    align-items: center;

    .box-search {
        position: relative;
        width: 50vw;
        border-radius: 5px;
        background: #fff;
        color: #000;
        overflow-y: auto;
        transform: scale(0.1);
        opacity: 0;
        animation: box-search-in 0.3s forwards;

        @keyframes box-search-in {
            0% {
                transform: scale(0.1);
                opacity: 0;
            }

            100% {
                transform: scale(1);
                opacity: 1;
            }
        }

        .close {
            color: red;
            position: absolute;
            top: 30px;
            right: 60px;
            font-size: 20px;
            cursor: pointer;
        }
    }
}

.btn-boolbnb {
    background: #ff385c;
    outline: none;
    color: #fff;
    border-radius: 25px;
    border: 1px solid transparent;
    transition: color 0.3s, background-color 0.4s, border-color 0.4s;
    &:hover {
        color: #ff385c;
        border-color: #ff385c;
        background: #fff;
    }
}

.badge-boolbnb {
    color: #ff385c;
    background: #fff;
    border: 1px solid #ff385c;
}

.arrow-container {
    position: absolute;
    left: -55px;
    top: 45px;
    .back-arrow {
        font-size: 35px;
        color: #ff385dad;
        cursor: pointer;
        transition: transform 0.3s;
        &:hover {
            transform: translateX(-10px);
            color: #ff385c;
        }
    }
}

.user-icon {
    font-size: 20px;
    margin-left: 9px;
}

.word-wrap {
    word-wrap: anywhere;
}
</style>
