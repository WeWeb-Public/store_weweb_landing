<template>
    <div class="mobile-store">
        <div class="dropdown-mobile">
            <div class="select">
                <select name="menus-select" v-model="selectedTheme" @change="selectTheme(selectedTheme)">
                    <option v-for="theme in themes" :key="theme.id" :value="theme.id">
                        <div>{{ theme.themeName }}</div>
                    </option>
                </select>
                <div class="text"></div>
                <i class="wwi wwi-chevron-down"></i>
            </div>
        </div>

        <!-- <div class="sections-wrapper">
            <div class="section" v-for="(section, index) in themes" :key="index" @click="selectSection(section)">
                <div class="section-wrapper" :style="{'background-image': 'url('+ getPreviewUrl(section)+')'}">
                    <div class="section-preview">
                        <div class="preview-name">{{ section.name }}</div>
                        <div class="preview-category">{{ section.categoryName }}</div>
                        <div class="preview-tag-wrapper">
                            <div class="preview-tag" v-for="(tag, index) in formatTags(section.tags)" :key="index">{{ '#' + tag }}</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>-->
    </div>
</template>

<script> 

export default {
    name: "wwMobileStore",

    props: {
        filtredThemes: Array,
    },
    computed: {
        themes() {
            return this.filtredThemes;
        },
    },
    data() {
        return {
            wwManagerLang: wwLib.wwManagerLang,
            search: '',
            stores: [],

            selectedStore: null,
            selectedCategory: null,
            selectedTheme: null,
            themeSection: null,
            filterThemes: null
        };
    },

    methods: {
        async init() {

            console.log('themes:', this.filtredThemes)
        },

        formatTags(tags) {
            try {
                let _tagsArray = []
                const tagsArray = []
                if (tags) {
                    _tagsArray = tags.split(',')
                    for (const tag of _tagsArray) {
                        tagsArray.push(tag.trim())
                    }
                }
                return tagsArray
            } catch (error) {
                console.error(error)
            }
        },
        searchFilter(section, search) {
            try {
                if (section.tags) {
                    return section.tags.includes(search)
                }
                else if (section.name) {
                    return section.name.includes(search)
                } else
                    return false
            } catch (error) {
                console.error(error)
                return false
            }
        },

        async selectTheme(theme) {
            console.log('theme:', theme)
            try {
                this.loading = true
                this.selectedTheme = theme
                this.storeSectionTypes = await this.getPublicStoreSectionTypes(theme.themeId)
                this.loading = false
                this.$forceUpdate();
            } catch (error) {
                console.error(error)
            }
        },



        async getPublicStoreSectionTypes(themeId) {
            try {
                const _url = `theme/${themeId}/get_public_store_sections`

                const _requestUrl = this.getApiUrl(_url)
                const response = await this.apiRequest(_url, 'GET')
                console.log('response:', response)
                return response.data
            } catch (error) {
                console.error(error)
            }

        },
        /* Get api url */
        getApiUrl(url) {
            // return 'localhost:3000/v1/' + url
            if (wwLib.envMode == 'development') {
                return 'http://localhost:3000/v1/' + url
                // return 'http://weweb-api:3000/v1/' + url
            } else if (wwLib.envMode == 'preprod') {
                return 'https://api.weweb.dev/v1/' + url
            } else if (wwLib.envMode == 'staging') {
                return 'https://api.weweb.space/v1/' + url
            } else {
                return 'https://api.weweb.app/v1/' + url
            }
        },

        /* - make api request to weweb-api - */
        async apiRequest(url, method) {
            try {
                const _requestUrl = this.getApiUrl(url)
                console.log('_requestUrl:', _requestUrl)
                const result = await axios({
                    method,
                    url: _requestUrl,
                    data: {}
                });
                if (!result) throw new Error("REQUEST_FAILED")
                return result

            } catch (error) {
                console.error(error)
            }

        },


        async selectCategory(category) {
            this.selectedCategory = category;
            this.$forceUpdate();
        },


        getPreviewUrl(section) {
            try {
                let url = 'https://i.twic.pics/v1/resize=800/'
                if (!section.previews.length) {
                    // url += wwLib.wwApiRequests._getCdnUrl() + 'public/images/no_preview.jpg';
                    url += "https://cdn.weweb.app/public/images/no_image_selected.png"

                } else {
                    if (section.previews[0].includes("http"))
                        url += section.previews[0];
                    else
                        // url += wwLib.wwApiRequests._getCdnUrl() + 'developers/' + section.previews[0]
                        url += "https://cdn.weweb.app/public/images/no_image_selected.png"

                }
                return url;
            } catch (error) {
                console.error(error)
            }
        },
        getStoreDisplayName(store) {
            try {
                if (this.storeNames[store.name]) {
                    return wwLib.wwManagerLang.getText(this.storeNames[store.name]);
                }
                return store.name;
            } catch (error) {
                console.error(error)
            }
        }

    },
    mounted: function () {
        this.init()
    },
    beforeDestroy() {

    }
};
</script>

<style scoped lang="scss">
@import "../assets/manager-css/main";
.mobile-store {
    width: 95%;
    .dropdown-mobile {
        pointer-events: all;
        // display: none;
        justify-content: center;
        width: 100%;
        .select {
            select {
                display: block;
                width: 100%;
                border: none;
                appearance: none;
                outline: none;
                // color: white;
                width: 100%;
                height: 100%;
                position: absolute;
                top: 0;
                left: 0;
                background: none;
                padding: inherit;
                padding: 2px 50px 2px 25px;
                cursor: pointer;
                font-size: 15px;
                font-weight: bold;
                line-height: 30px;
            }
            i {
                width: 25px;
                height: 25px;
                font-size: 25px;
            }
            padding: 2px 20px 2px 25px;
            position: relative;
            pointer-events: all;
            display: -webkit-box;
            display: -ms-flexbox;
            display: flex;

            justify-content: space-between;

            align-items: center;
            // color: white;
            margin: 20px 0;
            width: 250px;
            height: 60px;
            border-radius: 10px;
            cursor: pointer;
            background: linear-gradient(90deg, #455fa2 0%, #2d84c1 100%);
        }
    }
}
</style>


