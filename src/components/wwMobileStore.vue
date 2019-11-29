<template>
    <div class="mobile-store">
        <div class="dropdown-mobile">
            <div class="select">
                <select name="menus-select" v-model="selectedTheme" @change="selectTheme(selected)">
                    <option v-for="theme in themes" :key="theme.id" :value="theme.themeName" v-show="theme.sectionsCount > 0">
                        <div>{{ theme.themeName }}</div>
                    </option>
                </select>
                <div class="text" v-if="selectedTheme && selectedTheme.themeName">{{ selectedTheme.themeName }}</div>
                <i class="wwi wwi-chevron-down"></i>
            </div>
        </div>

        <div class="mobile-sections-wrapper">
            <div class="section" v-for="(section, index) in storeSectionTypes" :key="index">
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
        </div>
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
        selected() {
            for (const theme of this.themes) {
                if (this.selectedTheme === theme.themeName)
                    return theme
            }
            return undefined
        },
    },
    data() {
        return {
            dbCategories: null,
            filters: [],
            selectedSection: {},
            filterId: null,
            fixedCategories: null,
            storeSectionTypes: null,
            selectedTheme: undefined,
            loading: false,
            showSectionName: false,
            filterOptions: {
                type: 'text',
                values: [
                    {
                        value: null,
                        default: true,
                        text: {
                            en: 'Filters',
                            fr: 'Filters'
                        }
                    }
                ]
            },
        };
    },

    methods: {
        async init() {

            await this.getAssetsThemes()

            this.selectedTheme = this.filteredThemes[0]
        },
        async getAssetsThemes() {
            try {
                this.filters = await this.getMyAssetsThemes()

                if (this.filteredThemes && this.filteredThemes.length > 0) {

                    await this.selectTheme(this.filteredThemes[0])
                    await this.formatThemeForFilter(this.filters)
                } else {
                    console.error("no theme");
                }
            } catch (error) {
                console.error(error)
            }
        },

        /* get user themes, added origin to make the distinction between the theme of the design and the user */
        async getMyAssetsThemes() {
            try {
                const _url = 'theme/get_public_store_theme'

                const _usersThemes = await this.apiRequest(_url, 'GET')

                const _themes = []

                for (const theme of _usersThemes.data) {
                    _themes.push(theme)
                }

                return _themes
            } catch (error) {
                console.error(error)
            }
        },

        /* Get api url */
        getApiUrl(url) {
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
    width: 100%;
    min-height: 100px;
    @media (min-width: 1024px) {
        display: none;
    }
    display: flex;
    justify-content: center;
    flex-direction: column;
    pointer-events: all;
    .dropdown-mobile {
        pointer-events: all;
        display: flex;
        justify-content: center;
        width: 100%;
        .select {
            color: white;
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

    .mobile-sections-wrapper {
        flex-grow: 1;
        overflow-y: auto;
        flex-wrap: wrap;
        font-family: $ww-font;
        display: flex;
        box-sizing: content-box;
        justify-content: space-evenly;
        // justify-content: flex-start;
        // align-content: flex-start;
        .section {
            flex-basis: 45%;

            margin-bottom: 25px;
            cursor: pointer;
            border-radius: 2px;
            justify-content: center;
            align-items: center;

            .section-wrapper {
                position: relative;
                padding-bottom: 56.25%;
                border-radius: 2px;
                background-color: white;
                background-size: contain;
                background-position: center;
                background-repeat: no-repeat;
                border: 2px solid white;
                width: 100%;
                height: 100%;
                box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.15);
                &:hover {
                    box-shadow: 0 3px 6px 0 rgba(0, 0, 0, 0.17);
                    .section-preview {
                        visibility: visible;
                        opacity: 1;
                    }
                }
            }
            .section-preview {
                display: flex;
                flex-direction: column;
                position: absolute;
                min-height: 50px;
                bottom: 0;
                left: 0;
                right: 0;
                background: linear-gradient(
                    90deg,
                    rgba(29, 151, 127, 0.9) 0%,
                    rgba(44, 195, 183, 0.9) 100%
                );
                opacity: 0;
                padding: 5px 10px 10px 10px;
                color: white;
                visibility: hidden;
                border-radius: 0 0 3px 3px;
                transition: opacity 0.2s, visibility 0.2s;
                .preview-name {
                    text-transform: uppercase;
                    font-size: 1.2rem;
                    font-weight: bold;
                    margin-bottom: 10px;
                }
                .preview-category {
                    text-transform: uppercase;
                }
                .preview-tag-wrapper {
                    display: inline-flex;
                    font-size: 0.7rem;
                    margin-top: 5px;

                    .preview-tag {
                        margin-right: 5px;
                    }
                }
            }
        }
    }
}
</style>


