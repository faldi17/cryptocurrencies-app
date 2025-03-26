<template>
    <!-- ============ Marquee ============ -->
    <div class="marquee w-full h-12 overflow-hidden bg-black relative">
        <ul class="marquee-content flex h-full" ref="mq">
            <li
                v-for="coin in coins"
                :key="coin.id"
                class="flex items-center flex-shrink-0 text-white px-4"
            >
                <img :src="coin.image" alt="coin logo" class="coin-img" />
                <p class="font-bold">{{ coin.name }}</p>
                <p class="uppercase text-xs text-gray-300 ml-1">
                    ({{ coin.symbol }})
                </p>
                <p class="font-bold ml-auto">
                    ${{ coin.current_price.toLocaleString() }}
                </p>
                <p
                    :class="{
                        'text-green-400':
                            coin.price_change_percentage_1h_in_currency > 0,
                        'text-red-400':
                            coin.price_change_percentage_1h_in_currency < 0,
                    }"
                    class="font-bold ml-2"
                >
                    <fa
                        :icon="
                            coin.price_change_percentage_1h_in_currency > 0
                                ? 'caret-up'
                                : 'caret-down'
                        "
                        class="mr-1"
                    />
                    {{
                        coin.price_change_percentage_1h_in_currency.toFixed(2)
                    }}%
                </p>
            </li>
        </ul>
    </div>

    <!-- ============ Events ============ -->
    <section class="event">
        <div
            class="container mx-auto py-6 flex flex-wrap justify-center gap-6 px-4"
        >
            <a
                v-for="event in events"
                :key="event.id"
                :href="event.link"
                class="event-card"
                target="_blank"
            >
                <img
                    :src="event.image"
                    alt="event image"
                    class="event-image"
                    @error="handleImageError"
                />
                <div class="event-content">
                    <p class="event-title">{{ event.title }}</p>
                    <p class="event-desc">{{ event.description }}</p>
                </div>
            </a>
        </div>
    </section>

    <!-- ============ Table ============ -->
    <div class="container mx-auto pt-12 pb-32 px-4">
        <div class="flex justify-end pb-4">
            <input
                v-model="searchQuery"
                type="text"
                placeholder="Search..."
                class="border border-gray-400 rounded p-2 focus:outline-none focus:ring focus:ring-gray-300"
            />
        </div>

        <table class="w-full border border-gray-200">
            <thead class="bg-gray-100">
                <tr class="text-left text-gray-600 text-sm">
                    <th class="p-4">Name</th>
                    <th>Price</th>
                    <th>1h %</th>
                    <th class="hidden sm:table-cell">Market Cap</th>
                    <th class="hidden sm:table-cell">Volume</th>
                </tr>
            </thead>
            <tbody class="divide-y">
                <tr
                    v-for="coin in filteredCoins"
                    :key="coin.id"
                    class="text-sm hover:bg-gray-50 transition duration-300"
                >
                    <td class="p-4 flex items-center">
                        <img
                            :src="coin.image"
                            alt="coin logo"
                            class="coin-img"
                        />
                        <p class="font-bold ml-2">{{ coin.name }}</p>
                        <p class="uppercase text-gray-500 hidden sm:block ml-2">
                            ({{ coin.symbol }})
                        </p>
                    </td>
                    <td class="font-bold">
                        ${{ coin.current_price.toLocaleString() }}
                    </td>
                    <td class="font-bold">
                        <span
                            :class="{
                                'text-green-500':
                                    coin.price_change_percentage_1h_in_currency >
                                    0,
                                'text-red-500':
                                    coin.price_change_percentage_1h_in_currency <
                                    0,
                            }"
                        >
                            <fa
                                :icon="
                                    coin.price_change_percentage_1h_in_currency >
                                    0
                                        ? 'caret-up'
                                        : 'caret-down'
                                "
                                class="mr-1"
                            />
                            {{
                                coin.price_change_percentage_1h_in_currency.toFixed(
                                    2
                                )
                            }}%
                        </span>
                    </td>
                    <td class="hidden sm:table-cell">
                        ${{ coin.market_cap.toLocaleString() }}
                    </td>
                    <td class="hidden sm:table-cell">
                        {{ coin.total_volume.toLocaleString() }}
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
    import { ref, computed, onMounted } from "vue";
    import axios from "axios";

    export default {
        setup() {
            const coins = ref([]);
            const events = ref([]);
            const searchQuery = ref("");

            const getCoins = async () => {
                try {
                    const res = await axios.get(
                        "https://api.coingecko.com/api/v3/coins/markets",
                        {
                            params: {
                                vs_currency: "usd",
                                order: "market_cap_desc",
                                per_page: 100,
                                page: 1,
                                sparkline: false,
                                price_change_percentage: "1h",
                            },
                        }
                    );
                    coins.value = res.data;
                } catch (err) {
                    console.error("Error fetching coins:", err);
                }
            };

            const getEvents = async () => {
                try {
                    const res = await axios.get(
                        "https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=10&page=1&sparkline=false"
                    );
                    events.value = res.data.map((coin) => ({
                        id: coin.id,
                        title: coin.name + " Updates",
                        description: `Latest market update for ${
                            coin.name
                        }. Current price: $${coin.current_price.toLocaleString()}`,
                        image:
                            coin.image ||
                            "https://via.placeholder.com/320x200?text=No+Image",
                        link: `https://www.coingecko.com/en/coins/${coin.id}`,
                    }));
                } catch (err) {
                    console.error("Error fetching events:", err);
                }
            };

            const handleImageError = (event) => {
                event.target.src =
                    "https://via.placeholder.com/320x200?text=No+Image";
            };

            onMounted(() => {
                getCoins();
                getEvents();
                setInterval(getCoins, 60000);
            });

            const filteredCoins = computed(() =>
                coins.value.filter(
                    (coin) =>
                        coin.name
                            .toLowerCase()
                            .includes(searchQuery.value.toLowerCase()) ||
                        coin.symbol
                            .toLowerCase()
                            .includes(searchQuery.value.toLowerCase())
                )
            );

            return {
                coins,
                events,
                searchQuery,
                filteredCoins,
                handleImageError,
            };
        },
    };
</script>

<style scoped>
    /* MARQUEE  */
    .marquee {
        width: 100%;
        overflow: hidden;
        background: black;
        padding: 10px 0;
        position: relative;
        white-space: nowrap;
    }

    .marquee-content {
        display: flex;
        gap: 20px;
        min-width: fit-content;
        animation: scrolling 120s linear infinite;
    }

    .marquee-content:hover {
        animation-play-state: paused;
    }

    .marquee-content::after {
        content: attr(data-text);
        display: flex;
        gap: 20px;
    }

    @keyframes scrolling {
        from {
            transform: translateX(0);
        }
        to {
            transform: translateX(-50%);
        }
    }

    .coin-img {
        width: 24px;
        height: 24px;
        margin-right: 6px;
        object-fit: contain;
    }

    /* EVENTS */
    .event {
        background: #f9f9f9;
        padding: 4rem 0;
    }

    .event-card {
        width: 100%;
        max-width: 18rem;
        background: white;
        border-radius: 8px;
        overflow: hidden;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        transition: transform 0.3s;
        padding: 12px;
    }

    .event-card:hover {
        transform: translateY(-5px);
    }

    .event-image {
        width: 100%;
        height: 180px;
        object-fit: cover;
        border-radius: 6px;
    }

    .event-content {
        padding: 12px;
    }

    .event-title {
        font-size: 1rem;
        font-weight: bold;
        margin-bottom: 4px;
    }

    .event-desc {
        font-size: 0.875rem;
        color: #555;
    }

    @media (max-width: 640px) {
        .event {
            padding: 2rem 1rem;
        }

        .event-card {
            max-width: 100%;
        }
    }
</style>
