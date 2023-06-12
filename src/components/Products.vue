<script>
const API_URL_PRODUCTS = 'https://localhost:44333/api/Product/';
const API_URL_CHECKOUT = 'https://localhost:44333/api/Checkout/ProcessOrder';

export default {
    data() {
        return {
            products: [],
            error: null,
            order: {
                Name: '',
                Line1: '',
                Line2: '',
                Line3: '',
                City: '',
                State: '',
                GiftWrap: false,
            },
            showCheckout: false,
            orderSubmitted: false,
            cart: JSON.parse(window.localStorage.getItem('cart')) || [],
        };
    },
    computed: {
        cartTotal() {
            return this.cart.reduce((total, product) => total + product.Price, 0);
        }
    },
    mounted() {
        this.getProducts();
    },
    methods: {
        getProducts() {
            var request = new XMLHttpRequest();
            request.open('GET', API_URL_PRODUCTS, true);

            request.onload = () => {
                if (request.status >= 200 && request.status < 400) {
                    this.products = JSON.parse(request.responseText);
                } else {
                    this.error = 'Si è verificato un errore durante il caricamento dei prodotti.';
                }
            };

            request.onerror = () => {
                this.error = 'Si è verificato un errore durante il caricamento dei prodotti.';
            };

            request.send();
        },
        addToCart(product) {
            this.cart.push(product);
            window.localStorage.setItem('cart', JSON.stringify(this.cart));
        },
        removeFromCart(product) {
            this.cart = this.cart.filter(p => p.ProductID !== product.ProductID);
            window.localStorage.setItem('cart', JSON.stringify(this.cart));   
        },
        clearCart() {
            this.cart = [];
            window.localStorage.setItem('cart', JSON.stringify(this.cart));
        },
        submitOrder() {
            var request = new XMLHttpRequest();
            request.open('POST', API_URL_CHECKOUT, true);
            request.setRequestHeader('Content-Type', 'application/json');
            
            // Converti il carrello in un formato con quantità
            let cartWithQuantity = this.cart.map(item => {
                return { ProductID: item.ProductID, Quantity: 1 }; // Assumiamo che la quantità sia sempre 1
            });

            // Crea un nuovo oggetto che combina le informazioni dell'ordine e del carrello
            let orderWithCart = {
                ...this.order,
                CartItems: cartWithQuantity
            };

            request.onload = () => {
                if (request.status >= 200 && request.status < 400) {
                    this.orderSubmitted = true;
                    this.showCheckout = false;
                    this.clearCart();
                } else {
                    this.error = 'Si è verificato un errore durante l\'inserimento dell\'ordine.';
                }
            };

            request.onerror = () => {
                this.error = 'Si è verificato un errore durante l\'inserimento dell\'ordine.';
            };

            request.send(JSON.stringify(orderWithCart));

            //DEBUG
            console.log(JSON.stringify(orderWithCart));

        }

    }
}
</script>

<template>
    <div>
        <div v-if="error">
            <p>{{ error }}</p>
        </div>
        <div v-else>
            <h2>Prodotti</h2>
            <table>
                <tr>
                    <th>Nome</th>
                    <th>Descrizione</th>
                    <th>Categoria</th>
                    <th>Prezzo</th>
                    <th></th>
                </tr>
                <tr v-for="product in products" :key="product.ProductID">
                    <td>{{ product.Name }}</td>
                    <td>{{ product.Description }}</td>
                    <td>{{ product.Category }}</td>
                    <td>{{ product.Price }}</td>
                    <td><button @click="addToCart(product)">Aggiungi al carrello</button></td>
                </tr>
            </table>
            <h2>Carrello</h2>
            <table>
                <tr>
                    <th>Nome</th>
                    <th>Descrizione</th>
                    <th>Categoria</th>
                    <th>Prezzo</th>
                    <th></th>
                </tr>
                <tr v-for="product in cart" :key="product.ProductID">
                    <td>{{ product.Name }}</td>
                    <td>{{ product.Description }}</td>
                    <td>{{ product.Category }}</td>
                    <td>{{ product.Price }}</td>
                    <td><button @click="removeFromCart(product)">Rimuovi dal carrello</button></td>
                </tr>
            </table>
            <p>Totale: {{ cartTotal }}</p>
            <button @click="clearCart">Svuota Carrello</button>
            <button @click="showCheckout = true">Vai al checkout</button>
        </div>
        
        <!-- Form di checkout -->
        <div v-if="showCheckout">
            <h2>Checkout</h2>
                <form @submit.prevent="submitOrder">
                    <div>
                        <label for="name">Nome:</label>
                        <input id="name" v-model="order.Name" required>
                    </div>
                    <div>
                        <label for="line1">Indirizzo Linea 1:</label>
                        <input id="line1" v-model="order.Line1" required>
                    </div>
                    <div>
                        <label for="line2">Indirizzo Linea 2:</label>
                        <input id="line2" v-model="order.Line2">
                    </div>
                    <div>
                        <label for="line3">Indirizzo Linea 3:</label>
                        <input id="line3" v-model="order.Line3">
                    </div>
                    <div>
                        <label for="city">Città:</label>
                        <input id="city" v-model="order.City" required>
                    </div>
                    <div>
                        <label for="state">Stato:</label>
                        <input id="state" v-model="order.State" required>
                    </div>
                    <div>
                        <label for="giftWrap">Confezione regalo:</label>
                        <input id="giftWrap" type="checkbox" v-model="order.GiftWrap">
                    </div>
                <button type="submit">Effettua ordine</button>
            </form>
                <!-- Messaggio di conferma dell'ordine -->
            <div v-if="orderSubmitted">
                <p>Hai effettuato l'ordine</p>
            </div>
        </div>
    </div>
</template>



<style scoped>
    h2 {
        margin-top: 20px;
    }
    table {
        width: 100%;
        margin-bottom: 20px;
    }
    th {
        background-color: #f2f2f2;
        text-align: left;
    }
    td, th {
        padding: 10px;
    }
    button {
        margin-top: 10px;
    }
</style>
