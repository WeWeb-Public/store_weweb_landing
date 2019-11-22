<template>
    <div class="ww-popup-section-name">
        <div class="section-header">- {{ section.name }} -</div>
        <div class="section-wrapper">
            <div class="preview-wrapper">
                <div class="image-previews ww-scroll-bar">
                    <div class="laptop-preview-container">
                        <div class="laptop-preview" :style="{'background-image': 'url('+getPreviewUrl(section.previews, 600)+')'}"></div>
                        <!-- <div class="preview-mask">
                            <wwManagerButton class="button" color="pink-gradient" center="true" invert="true" @click="selectPcPreview()">EDIT</wwManagerButton>
                        </div>-->
                    </div>
                    <div class="mobile-preview-container">
                        <div class="mobile-preview" :style="{'background-image': 'url('+getPreviewUrl(section.previewsMobile, 300)+')'}"></div>
                    </div>
                </div>
            </div>
        </div>
        <div class="close-button-wrapper">
            <div class="close-btn" @click="closeSection()">Close Preview</div>
        </div>
    </div>
</template>

<script> 

export default {
    name: "wwPopupSectionNamePreview",
    props: {
        section: {
            type: Object,
            default: function () {
                return {}
            }
        },
    },
    data() {
        return {
            sectionName: '',
            sectionType: {},
            tags: '',

        };
    },


    methods: {
        init() {
            console.log('this.section:', this.section)
            // this.sectionType = this.options.data.sectionType
            // this.sectionName = this.sectionType.name
        },

        closeSection() {
            console.log('close:')
            this.$emit('close')
        },

        getPreviewUrl(previews, size) {
            console.log('previews:', previews)
            try {
                let url = ''
                // let url = `https://i.twic.pics/v1/resize=${size}/`
                if (!previews || !previews.length) {
                    url += wwLib.wwApiRequests._getCdnUrl() + 'public/images/no_preview.jpg';
                } else {
                    if (previews[0].includes("http"))
                        url += previews[0];
                    else
                        url += wwLib.wwApiRequests._getCdnUrl() + 'developers/' + previews[0]
                }
                return url;
            } catch (error) {
                console.error(error)
            }
        },
        // getPreviewUrl(previews, size) {
        //     try {
        //         let url = ''
        //         // let url = `https://i.twic.pics/v1/resize=${size}/`
        //         if (!previews || !previews.length) {
        //             // url += wwLib.wwApiRequests._getCdnUrl() + 'public/images/no_preview.jpg';
        //             url += "https://cdn.weweb.app/public/images/no_image_selected.png"

        //         } else {
        //             if (previews[0].includes("http"))
        //                 url += previews[0];
        //             else
        //                 // url += wwLib.wwApiRequests._getCdnUrl() + 'developers/' + previews[0]
        //                 url += "https://cdn.weweb.app/public/images/no_image_selected.png"

        //         }
        //         return url;
        //     } catch (error) {
        //         console.error(error)
        //     }
        // },


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

.ww-popup-section-name {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(255, 255, 255, 0.95);
    z-index: 100;
    pointer-events: all;

    .section-header {
        box-sizing: border-box;
        width: 100%;
        position: relative;
        flex-basis: 100px;
        display: flex;

        justify-content: center;
        background-color: white;
        color: #e02a4d;
        font-family: "Montserrat", sans-serif;
        font-size: 1.6rem;
        text-transform: uppercase;
        font-weight: bold;
        padding: 50px 10px 0 10px;
    }
    .section-wrapper {
        display: flex;
        flex-direction: column;
        justify-content: center;
        font-family: $ww-font;
        width: 95%;
        color: $ww-grey-light;
        overflow: hidden;
        margin-top: 50px;
        @media (min-width: 1200px) {
            flex-direction: row;
            width: 100%;
        }

        .preview-wrapper {
            display: flex;
            flex-direction: column;
            justify-content: center;
            font-family: $ww-font;
            display: inline-flex;
            flex-direction: column;
            width: 100%;
            @media (min-width: 992px) {
                width: 70%;
            }
            .image-previews {
                margin-bottom: 20px;
                .laptop-preview-container {
                    width: 75.96439169%;
                    float: left;
                    padding-right: 10px;
                    .laptop-preview {
                        box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.15);
                        border: 2px solid white;
                        width: 100%;
                        padding-bottom: 56.25%;
                        background-size: contain;
                        background-position: center;
                        background-repeat: no-repeat;
                        &:hover {
                            box-shadow: 0 3px 6px 0 rgba(0, 0, 0, 0.17);
                        }
                    }
                }
                .mobile-preview-container {
                    float: left;
                    width: 24.03561%;
                    height: 100%;
                    .mobile-preview {
                        background-size: contain;
                        background-position: 50%;
                        background-repeat: no-repeat;
                        border: 1px solid #fff;
                        box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.15);
                        width: 100%;
                        height: 100%;
                        padding-bottom: 176.25%;
                        &:hover {
                            box-shadow: 0 3px 6px 0 rgba(0, 0, 0, 0.17);
                        }
                    }
                }
            }

            // .preview-wrapper {
            //     display: flex;
            //     flex-direction: column;
            //     justify-content: flex-start;
            //     font-family: $ww-font;
            //     display: inline-flex;
            //     flex-direction: column;
            //     width: 100%;
            //     @media (min-width: 992px) {
            //         width: 60%;
            //     }
            //     .image-previews {
            //         margin-bottom: 20px;
            //         display: flex;
            //         .laptop-preview-container {
            //             width: 75.96439169%;
            //             float: left;
            //             padding-right: 30px;
            //             .preview-mask {
            //                 opacity: 0;
            //             }
            //             &:hover {
            //                 cursor: pointer;
            //                 .preview-mask {
            //                     opacity: 1;
            //                     width: fit-content;
            //                     position: relative;
            //                     top: -50%;
            //                     left: 50%;
            //                     transform: translate(-50%, -50%);
            //                 }
            //             }
            //             .laptop-preview {
            //                 box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.15);
            //                 border: 2px solid white;
            //                 width: 100%;
            //                 padding-bottom: (9/16) * 100%;
            //                 background-size: contain;
            //                 background-position: center;
            //                 background-repeat: no-repeat;
            //                 &:hover {
            //                     box-shadow: 0 3px 6px 0 rgba(0, 0, 0, 0.17);
            //                 }
            //             }
            //         }
            //         .mobile-preview-container {
            //             float: left;
            //             width: 100% - 75.96439169%;
            //             height: 100%;
            //             .mobile-preview {
            //                 background-size: contain;
            //                 background-position: center;
            //                 background-repeat: no-repeat;
            //                 border: 1px solid white;
            //                 box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.15);
            //                 width: 100%;
            //                 height: 100%;
            //                 &:hover {
            //                     box-shadow: 0 3px 6px 0 rgba(0, 0, 0, 0.17);
            //                 }
            //             }
            //         }
            //     }
        }
    }
    .close-button-wrapper {
        width: 100%;
        display: flex;
        justify-content: center;
        margin-top: 30px;
        .close-btn {
            box-sizing: border-box;
            padding: 5px 10px;
            border-radius: 20px;
            text-transform: uppercase;
            color: white;
            font-size: 13px;
            font-weight: bold;
            font-family: "Montserrat", sans-serif;
            cursor: pointer;
            right: 30px;
            border: 0px;
            background-color: $ww-blue;
        }
    }
}

.preview {
    padding: 30px;
    text-align: center;

    @media (min-width: 992px) {
        flex-basis: 400px;
    }

    @media (min-width: 1200px) {
        flex-basis: 800px;
    }

    img {
        width: 100%;
        border: 1px solid $ww-grey-light;
    }

    .btn {
        display: inline-block;
        margin-top: 20px;
    }
}
</style>
