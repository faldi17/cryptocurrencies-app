<template>
    <!-- ============ Marquee ============ -->
    <div class="marquee w-full h-12 overflow-hidden bg-black relative">
        <ul class="marquee-content flex h-full" ref="mq">
            <li
                v-for="coin in coins"
                :key="coin.id"
                class="flex justify-center items-center flex-shrink-0 text-white transform scale-75 lg:scale-100 px-4"
            >
                <div class="flex justify-between w-48">
                    <div class="flex items-center">
                        <img
                            :src="coin.image"
                            alt="coin logo"
                            class="w-4 h-4 lg:w-6 lg:h-6 rounded-full mr-4 object-cover"
                        />
                        <div class="hidden lg:block">
                            <p class="font-bold">{{ coin.name }}</p>
                            <p class="text-xs uppercase tracking-widest">
                                {{ coin.symbol }}
                            </p>
                        </div>
                    </div>
                    <div>
                        <p
                            class="font-bold text-xs lg:text-base flex justify-end"
                        >
                            ${{ coin.current_price.toLocaleString() }}
                        </p>
                        <p
                            :class="{
                                'text-green-400':
                                    coin.price_change_percentage_1h_in_currency >
                                    0,
                                'text-red-400':
                                    coin.price_change_percentage_1h_in_currency <
                                    0,
                            }"
                            class="font-bold text-xs flex justify-end items-center"
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
                        </p>
                    </div>
                </div>
            </li>
        </ul>
    </div>

    <!-- ============ Events ============ -->
    <section class="event">
        <div
            class="container mx-auto py-4 lg:py-24 flex flex-wrap justify-center gap-6"
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
    <div class="container mx-auto pt-18 pb-48 px-2">
        <div class="flex justify-end pb-2">
            <div class="relative">
                <input
                    v-model="searchQuery"
                    type="text"
                    placeholder="Search..."
                    class="border border-gray-500 rounded p-2 focus:outline-none focus:ring focus:ring-gray-300"
                />
                <fa
                    icon="search"
                    class="text-gray-500 absolute right-2 top-2/4 transform -translate-y-1/2"
                />
            </div>
        </div>

        <table class="table-fixed cursor-pointer">
            <thead class="bg-gray-200">
                <tr class="text-left text-gray-600 text-sm">
                    <th class="w-1/4 p-4">Name</th>
                    <th class="w-1/4">Price</th>
                    <th class="w-1/4">1h %</th>
                    <th class="w-1/4 hidden sm:table-cell">Market Cap</th>
                    <th class="w-1/4 hidden sm:table-cell">Volume</th>
                </tr>
            </thead>
            <tbody class="divide-y">
                <tr
                    v-for="coin in filteredCoins"
                    :key="coin.id"
                    class="text-sm hover:bg-gray-100 transition duration-300"
                >
                    <td class="p-4 flex items-center">
                        <img
                            :src="coin.image"
                            alt="coin logo"
                            class="w-6 h-6 rounded-full mr-1"
                        />
                        <p class="font-bold mr-1">{{ coin.name }}</p>
                        <p class="uppercase text-gray-500 hidden sm:table-cell">
                            {{ coin.symbol }}
                        </p>
                    </td>
                    <td class="font-bold">
                        ${{ coin.current_price.toLocaleString() }}
                    </td>
                    <td class="font-bold">
                        <div
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
                        </div>
                    </td>
                    <td class="hidden sm:table-cell">
                        ${{ coin.market_cap.toLocaleString() }}
                    </td>
                    <td class="pr-4 hidden sm:table-cell">
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
        name: "App",
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
                    console.log("Events Data (From Coins API):", res.data);

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

            return { coins, events, searchQuery, filteredCoins };
        },
    };
</script>

<style scoped>
    .marquee-content {
        display: flex;
        animation: scrolling 20s linear infinite;
    }

    .marquee-content:hover {
        animation-play-state: paused;
    }
    
    @keyframes scrolling {
        0% {
            transform: translateX(0);
        }
        100% {
            transform: translateX(-100%);
        }
    }

    .event a {
        transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
    }

    .event a:hover {
        transform: scale(1.05);
        box-shadow: 0px 5px 15px rgba(0, 0, 0, 0.2);
    }

    .event {
        background: #f9f9f9;
        padding: 4rem 0;
    }

    .event-card {
        width: 18rem;
        background: #fff;
        border-radius: 8px;
        overflow: hidden;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        transition: transform 0.3s ease, box-shadow 0.3s ease;
    }

    .event-card:hover {
        transform: translateY(-5px);
        box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
    }

    .event-image {
        width: 100%;
        height: 180px;
        object-fit: cover;
        background: #eee;
    }

    .event-content {
        padding: 1rem;
    }

    .event-title {
        font-size: 1rem;
        font-weight: bold;
        color: #333;
    }

    .event-desc {
        font-size: 0.875rem;
        color: #666;
        margin-top: 0.5rem;
    }
</style>
