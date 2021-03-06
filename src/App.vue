<template>
    <div id="app"
         class="container-fluid pr-0 pl-0"
         style="overflow-y: hidden; overflow-x: hidden">
        <div class="row no-gutters">
            <div id="side-bar">
                <SideBar/>
            </div>
            <div id="drag-sidebar" class="dragbar" @mousedown="dragSideBarDown"></div>
            <div id="stage">
                <Stage/>
            </div>
            <div id="drag-resultbar" class="dragbar" @mousedown="dragResultBarDown"
                 style="color: var(--stacker-background-alternative-color);"></div>
            <div id="result">
                <Result/>
            </div>
        </div>
    </div>
</template>

<script>

    const fs = window.remote.require('fs');
    import SideBar from "./components/sidebar/SideBar";
    import Result from "./components/result/Result";
    import Stage from "./components/stage/Stage";

    export default {
        components: {Stage, Result, SideBar},
        created() {
            window.ipcRenderer.on('dirnames', (event, asarDirectory, homeDirectory) => {
                process.chdir(homeDirectory);
                asarDirectory = !!asarDirectory ? asarDirectory + '/../': './';
                homeDirectory = homeDirectory + '/';
                this.$store.commit('createDotStackerDirectory', {
                    asarDirectory, homeDirectory
                });
                this.$store.commit('openRequisitionFile', {
                    router: this.$router,
                    file: homeDirectory + '.stacker/openRequisitions/examples.stk'
                });
            });

            window.ipcRenderer.on('clipboardCopy', () => this.$store.commit('clipboardCopy'));
            window.ipcRenderer.on('clipboardPaste', () => this.$store.commit('clipboardPaste'));
        },
        data() {
            return {
                dragging: false
            }
        },
        methods: {
            dragSideBarDown(event) {
                event.preventDefault();

                this.dragging = true;
                const stage = $('#stage');
                const ghostBar = $('<div>',
                    {
                        id: 'ghost-bar',
                        css: {
                            height: stage.outerHeight(),
                            top: stage.offset().top - 135 + 'px',
                            left: stage.offset().left
                        }
                    }).appendTo('body');

                $(document).mousemove(function (event) {
                    ghostBar.css("left", Math.min(Math.max(event.pageX + 2, 350), 500));
                });
                $(document).mouseup((event) => {
                    if (this.dragging) {
                        const eventX = Math.min(Math.max(event.pageX + 2, 350), 500);
                        const resultWidth = parseInt($('#result').css('width'));

                        const sideBarPercentage = ((eventX - 2) / window.innerWidth) * 100;
                        const resultPercentage = (resultWidth / window.innerWidth) * 100;
                        const stagePercentage = 100 - sideBarPercentage - resultPercentage;

                        $('#side-bar').css({
                            flex: `0 0 calc(${sideBarPercentage}% - 2px)`,
                            'max-width': `calc(${sideBarPercentage}% - 2px)`
                        });
                        $('#stage').css({
                            flex: `0 0 calc(${stagePercentage}% - 2px)`,
                            'max-width': `calc(${stagePercentage}% - 2px)`
                        });
                        $('#ghost-bar').remove();

                        $(document).unbind('mousemove');
                        $(document).unbind('mouseup');

                        this.dragging = false;
                    }
                });
            },
            dragResultBarDown(event) {
                event.preventDefault();

                this.dragging = true;
                const result = $('#result');
                const ghostBar = $('<div>',
                    {
                        id: 'ghost-bar',
                        css: {
                            height: result.outerHeight(),
                            top: result.offset().top,
                            left: result.offset().left
                        }
                    }).appendTo('body');

                $(document).mousemove(function (event) {
                    ghostBar.css("left", Math.max(Math.min(event.pageX + 2, window.innerWidth - 350), window.innerWidth - 600));
                });
                $(document).mouseup((event) => {
                    if (this.dragging) {
                        const eventX = Math.max(Math.min(event.pageX + 2, window.innerWidth - 350), window.innerWidth - 600);

                        const sideBarWidth = parseInt($('#side-bar').css('width'));
                        const sideBarPercentage = ((sideBarWidth + 2) / window.innerWidth) * 100;
                        const stagePercentage = ((eventX / window.innerWidth) * 100) - sideBarPercentage;
                        const resultPercentage = ((window.innerWidth - eventX - 2) / window.innerWidth) * 100;

                        $('#stage').css({
                            flex: `0 0 ${stagePercentage}%`,
                            'max-width': `${stagePercentage}%`
                        });
                        $('#result').css({
                            flex: `0 0 calc(${resultPercentage}% - 2px)`,
                            'max-width': `calc(${resultPercentage}% - 2px)`
                        });
                        $('#ghost-bar').remove();

                        $(document).unbind('mousemove');
                        $(document).unbind('mouseup');

                        this.dragging = false;
                    }
                });
            },
        }
    }
</script>

<style>
    :root {
        /*https://coolors.co/9fb630-ffe8af-c4bcf2-d4f2db-cef7a0*/
        --requisition-color: #FFE6A7;
        --publisher-color: #D4F2DB; /*#5e64c3; /*#6A8D73*/;
        --subscription-color: #C4BCF2; /*c86ba6;*/
        /*--passing-test-color: #42ba84;*/
        --passing-test-color: #9FB630;
        --failing-test-color: #a9524a;
        --invalid-color: #a9524a;
        /*--information-color: #7ed221;*/
        --enqueuer-color: #9FB630;

        --text-color: #eeeeee;
        --text-smooth-color: #a4a4a4;
        --stacker-header-background-color: #2b2a2e;
        --stacker-background-color: #353535;
        --stacker-background-alternative-color: #424242;
    }

    .dropdown-menu {
        /*background-color: var(--text-smooth-color);*/
        padding-bottom: 0;
        width: max-content;
        padding-top: 1px;
    }

    .dropdown-item {
        font-size: 14px;
        height: 34px;
        padding-left: 15px;
        font-weight: lighter;
        color: var(--text-color);
        background-color: var(--stacker-background-alternative-color);
    }

    .dropdown-item.active, .dropdown-item:active, .dropdown-item-active {
        text-decoration: none;
        background-color: var(--stacker-header-background-color);
        color: var(--text-smooth-color);
    }

    .dropdown-item:focus, .dropdown-item:hover {
        /*border-bottom: 2px solid var(--enqueuer-color);*/
        color: var(--text-color);
        background-color: var(--stacker-background-color);
    }

    .dropdown-item-disabled, .dropdown-item-disabled:hover, .dropdown-item-disabled.hover {
        text-decoration: none;
        pointer-events: none;
        background-color: var(--stacker-header-background-color);
        color: var(--text-smooth-color);
    }

    .dropdown-divider {
        height: 0;
        margin: .01rem 0;
        overflow: hidden;
    }

    .dropdown-header {
        padding: .1rem .5rem;
        font-size: .75em;
        color: var(--text-smooth-color);
        background-color: var(--stacker-header-background-color);
    }

    .tooltip-inner {
        max-width: none;
        color: var(--text-color);
        /*opacity: 0.9;*/
        text-align: left;
        border: 2px solid var(--stacker-background-alternative-color);
        background-color: var(--stacker-header-background-color);
        border-radius: .25rem;
    }

    .bs-tooltip-top .arrow::before {
        border-top-color: var(--stacker-background-alternative-color);
    }

    .invalid-input {
        box-shadow: 0 0 15px var(--invalid-color);
    }

    .invalid-name {
        text-decoration: underline wavy var(--invalid-color);
    }

    .stacker {
        background-color: var(--stacker-background-color);
    }

    .stacker-header {
        height: 150px;
        background-color: var(--stacker-header-background-color);
    }

    .stacker-label {
        font-size: 0.8em;
        color: var(--text-smooth-color);
    }

    .stacker-input {
        background-color: transparent;
        border-color: var(--stacker-background-alternative-color);
        color: var(--text-color);
        height: 32px;
        font-weight: lighter;
        font-size: 14px;
    }

    .stacker-input:focus {
        background-color: transparent;
        color: var(--text-color);
        box-shadow: 0 0 5px var(--enqueuer-color);
        /*border-color: #80bdff;*/
        border-color: var(--enqueuer-color);
    }

    .stacker-input:hover {
        border-color: var(--enqueuer-color);
        box-shadow: 0 0 5px var(--enqueuer-color);
    }

    #side-bar {
        flex: 0 0 calc(22% - 2px);
        max-width: calc(22% - 2px);
    }

    #stage {
        flex: 0 0 48%;
        max-width: 48%;
    }

    #result {
        flex: 0 0 calc(30% - 2px);
        max-width: calc(30% - 2px);
    }

    .dragbar {
        background-color: var(--stacker-header-background-color);
        float: right;
        width: 2px;
        cursor: col-resize;
    }

    #ghost-bar {
        width: 1px;
        background-color: var(--text-smooth-color);
        opacity: 0.5;
        position: absolute;
        cursor: col-resize;
        z-index: 999
    }

    .stacker-icon {
        cursor: pointer;
        color: var(--text-smooth-color);
        transform: scale(0.85);
        transition: 100ms ease transform;
    }

    .stacker-icon:hover, .stacker-icon.hover {
        transform: scale(1) rotate(7.5deg);
        color: var(--enqueuer-color);
    }

    .scroll-div {
        display: block;
        max-height: 100%;
        overflow-y: auto;
    }

    ::-webkit-scrollbar {
        background-color: transparent;
        width: .8em
    }

    ::-webkit-scrollbar {
        width: 8px; /* 1px wider than Lion. */
        /* This is more usable for users trying to click it. */
        background-color: rgba(0, 0, 0, 0);
        -webkit-border-radius: 100px;
    }

    /* hover effect for both scrollbar area, and scrollbar 'thumb' */
    ::-webkit-scrollbar:hover {
        background-color: rgba(0, 0, 0, 0.09);
    }

    /* The scrollbar 'thumb' ...that marque oval shape in a scrollbar */
    ::-webkit-scrollbar-thumb:vertical {
        /* This is the EXACT color of Mac OS scrollbars.
        Yes, I pulled out digital color meter */
        background: rgba(0, 0, 0, 0.5);
        -webkit-border-radius: 100px;
    }

    ::-webkit-scrollbar-thumb:vertical:active {
        background: rgba(0, 0, 0, 0.61); /* Some darker color when you click it */
        -webkit-border-radius: 100px;
    }
</style>
