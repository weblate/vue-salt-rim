<template>
    <div class="list-print-container">
        <h3>{{ $t('your-shopping-list') }}</h3>
        <ul>
            <li v-for="ingredient in list">
                <div class="checkbox"></div>
                <div class="name">
                    {{ ingredient.name }}
                </div>
            </li>
        </ul>
        <div class="list-print-footer">
            Generated by Bar Assistant
        </div>
    </div>
</template>
<script>
import ApiRequests from '@/ApiRequests';

export default {
    data() {
        return {
            list: {},
            printReady: false
        }
    },
    created() {
        window.addEventListener('afterprint', (e) => {
            window.close();
        });

        ApiRequests.fetchIngredients({'filter[on_shopping_list]': true, per_page: 500}).then(data => {
            this.list = data
            this.printReady = true
        }).catch(e => {
            this.$toast.error(e.message);
        })
    }
}
</script>

<style scoped>
.list-print-container {
    color: #000;
    font-size: 0.7rem;
    border: 1px dotted #000;
    padding: 10px;
    background-color: #fff;
    margin: 20px auto;
    max-width: 700px;
}

.list-print-container ul {
    list-style: none;
    margin: 0;
    padding: 0;
    display: grid;
    grid-template-columns: 1fr 1fr;
}

.list-print-container ul li {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 10px;
}

h3 {
    border-bottom: 3px double #333;
    text-align: center;
    font-family: var(--font-heading);
    font-size: 1.5rem;
    margin-bottom: 20px;
}

.checkbox {
    width: 25px;
    height: 25px;
    border: 2px solid #000;
    flex-grow: 0;
    flex-shrink: 0;
}

.list-print-footer {
    text-align: center;
    font-size: 0.6rem;
    margin-top: 10px;
}
</style>
