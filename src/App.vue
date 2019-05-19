<template>
    <div class="autocomplete">
        <input class="input-search" type="text" @input="onChange" v-model="search" @keyup.down="onArrowDown()" @keyup.up="onArrowUp" @keyup.enter="onEnter"/>
        <ul id="autocomplete-results" v-show="isOpen" class="autocomplete-results">

            <li class="loading" v-if="isLoading">
                Loading results...
            </li>

            <div v-else>
                <li class="titleResults">
                    Endereços
                </li>
                <li v-for="(result,index) in results.address" :key="result" @mouseover="setOver(result)" @click="setResult(result)" class="autocomplete-result" :class="{ 'is-active': index === arrowCounter }">
                    {{result}}
                </li>
                <li class="titleResults">
                    Bairros
                </li>
                <li v-for="(result) in results.neighborhood" :key="result" @mouseover="setOver(result)" @click="setResult(result)" class="autocomplete-result" :class="{ 'is-active': result === arrowCounter }">
                    {{result}}
                </li>
                <li class="titleResults">
                    Cidades
                </li>
                <li v-for="(result) in results.city" :key="result" @mouseover="setOver(result)" @click="setResult(result)" class="autocomplete-result" :class="{ 'is-active': result=== arrowCounter }">
                    {{result}}
                </li>

            </div>
        </ul>
        <pre>
      {{$data}}
    </pre>
    </div>
</template>
<script>
    import Axios from 'axios'

    export default {
        name: "autocomplete",
        data() {
            return {
                isOpen: false,
                results: [],
                search: "",
                over: "",
                isLoading: false,
                arrowCounter: 0,
                isAsync: false,
            };
        },
        methods: {
            onChange() {
                // Let's warn the parent that a change was made
                this.$emit("input", this.search);

                // Is the data given by an outside ajax request?
                if (this.isAsync) {
                    this.isLoading = true;
                } else {
                    // Let's search our flat array
                    this.filterResults();
                    this.isOpen = true;
                }
            },
            filterResults() {
                Axios.get('http://idata-api.local/api/search-locals.json?search=' + this.search).then((response) => {
                    if (response.data) {
                        this.results = response.data.registries
                    }
                })
            },
            setOver(result){
                this.over = result;
            },
            setResult(result) {
                if(result)
                this.search = result;
                this.isOpen = false;
            },
            onArrowDown() {
                if (this.arrowCounter < this.results.length) {
                    this.arrowCounter = this.arrowCounter + 1;
                }
            },
            onArrowUp() {
                if (this.arrowCounter > 0) {
                    this.arrowCounter = this.arrowCounter - 1;
                }
            },
            onEnter() {
                this.search = this.over;
                this.isOpen = false;
                this.arrowCounter = -1;
            },
            handleClickOutside(evt) {
                if (!this.$el.contains(evt.target)) {
                    this.isOpen = false;
                    this.arrowCounter = -1;
                }
            }
        },
        watch: {
            results: function (val, oldValue) {
                // actually compare them
                if (val.length !== oldValue.length) {
                    this.results = val;
                    this.isLoading = false;
                }
            }
        },
        mounted() {
            document.addEventListener("click", this.handleClickOutside);
        },
        destroyed() {
            document.removeEventListener("click", this.handleClickOutside);
        }
    }
</script>
<style>
    #app {
        font-family: "Avenir", Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        color: #2c3e50;
        margin-top: 60px;
    }

    .autocomplete {
        position: relative;
        width: 100%;
    }

    .titleResults{
        background-color: #094b0f;
        vertical-align: center;
        color: #e8e8e8;
        height: 30px;
        padding: 2px;
    }

    .input-search {
        width: 100%;
        background: #383838;
        color: white;
        height: 30px;
        border: none;
        padding: 3px;
    }

    .autocomplete-results {
        padding: 0;
        margin: 0;
        border: 1px solid #eeeeee;
        max-height: 400px;
        overflow: auto;
        width: 100%;
    }

    .autocomplete-result {
        list-style: none;
        text-align: left;
        padding: 4px 2px;
        cursor: pointer;
        border-bottom: 1px solid #e1e1e1;
    }

    .autocomplete-result.is-active,
    .autocomplete-result:hover {
        background-color: rgba(12, 96, 18, 0.65);
        color: white;
    }

</style>
