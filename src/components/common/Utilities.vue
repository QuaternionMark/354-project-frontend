<script>
    import Requests from "./Requests";
    const bcrypt = require("bcryptjs");
    const BCRYPT_SALT = "$2a$10$ssTHsnejHc6RrlyVbiNQ/O";

    const utils = {
        EMAIL_PATTERN: /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/,
        PASSWORD_PATTERN: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^a-zA-Z\d]).*$/,
        MAX_NAME_CHARACTERS: 30,
        name: "Utilities",
        data: () => ({
            PRIMARY_COLOR: "#00838F",
            ACCENT_COLOR: "#FF8F00",
            BASIC_GREY: "#9E9E9E"
        }),
        methods: {
            // Return to the previous page if one existed.
            goBack() {
                if (!utils.isEmpty(this.$route.meta.prevRoute.query.redirect)) {
                    this.$router.push("/home");
                } else {
                    window.history.length > 1 ? this.$router.go(-1) : this.$router.push("/");
                }
            },
            // Return to requested redirect, otherwise homepage.
            returnToRedirect() {
                let retPath = this.$route.query.redirect;
                if (!utils.isEmpty(retPath)) {
                    this.$router.push("/" + retPath);
                } else {
                    this.$router.push("/home");
                }
            },
            confirmBeforeGoBack() {
                let confirmed = confirm("Are you sure you want to exit?");
                if (confirmed) {
                    this.goBack();
                }
            },
            // Returns the route a given page should go to in order to logout and return to the same page.
            getLogoutRouter() {
                return "/logout?redirect=" + this.$route.path.substr(1);
            },
            // Returns the route a given page should go to in order to login and return to the same page.
            getLoginRouter() {
                return "/login?redirect=" + this.$route.path.substr(1);
            },
            getUserData(key) {
                return this.$store.state.currUser[key];
            },

            hashString(plainText) {
                try {
                    return bcrypt.hashSync(plainText, BCRYPT_SALT);
                } catch (error) {
                    return "";
                }
            },

            async updateShoppingCartAsync() {
                this.$store.commit("startCartLoad");

                let response = await Requests.getShoppingCartAsync();

                if (!response.error) {
                    if (response.status === Requests.HttpStatus.SUCCESS) {
                        this.$store.commit("setShoppingCart", response.data);
                    } else if (response.status === Requests.HttpStatus.BAD_REQUEST) {
                        this.$store.commit("setShoppingCart", null);
                    }
                    this.$store.commit("stopCartLoad");
                }
            },
            async updateCartQuantityAsync(item, newQuantity) {
                this.$store.commit("startCartLoad");

                let jsonData = {
                    productId: item.product.id,
                    quantity: Number(newQuantity)
                };

                let response = await Requests.updateCartLineAsync(jsonData);
                if (!response.error) {
                    await this.updateShoppingCartAsync();
                }
            }
        },

        // Misc. free functions.
        clone(object) {
            return JSON.parse(JSON.stringify(object));
        },
        isEmpty(object) {
            return object === undefined || object === null || object.length === 0 || object === "";
        },
        dictToQueryString(object) {
            return "?" + Object.keys(object).map(key => key + "=" + object[key]).join("&");
        }
    };

    export default utils;
</script>

<style scoped>

</style>
