<template>
  <div class="section has-background-dark" v-cloak>
    <div class="container has-text-white" style="margin-bottom: 40px;">
      Sort shipments:
      <div class="dropdown">
        <div class="dropdown-trigger">
          <button class="button" aria-haspopup="true" aria-controls="dropdown-menu">
            <span>{{ sort | humanizeSort }}</span>
            <span class="icon is-small">
              <i class="fas fa-angle-down" aria-hidden="true"></i>
            </span>
          </button>
        </div>
        <div class="dropdown-menu" id="dropdown-menu" role="menu">
          <div class="dropdown-content">
            <template v-for="available_sort in available_sorts">
              <a href="#" :data-new-sort="available_sort" :class="['dropdown-item', (available_sort == sort) ? 'is-active' : '']">
                {{ available_sort | humanizeSort }}
              </a>
            </template>
          </div>
        </div>
      </div>

    </div>

    <template v-if="loading">
      <div class="box">&nbsp;</div>
      <div class="pt-50 pb-50">&nbsp;</div>
    </template>
    <template v-else>
      <div class="container is-fluid">
        <div class="columns is-multiline">
          <template v-for="shipment_offer in shipment_offers">
            <div class="column is-one-third">
              <div class="card large">
                <div class="card-content">
                  <div class="media">
                    <div class="media-content">
                      <template v-if="sort === 'destination'">
                        <p class="title is-4 no-padding has-background-grey-lighter">
                          {{ shipment_offer.destination.city }}, {{ shipment_offer.destination.state }} &larr;
                          <br/>
                          {{ shipment_offer.origin.city }}, {{ shipment_offer.origin.state }}
                        </p>
                      </template>
                      <template v-else>
                        <p :class="['title', 'is-4', 'no-padding', (sort === 'origin') ? 'has-background-grey-lighter' : '']">
                          {{ shipment_offer.origin.city }}, {{ shipment_offer.origin.state }} &rarr;
                          <br/>
                          {{ shipment_offer.destination.city }}, {{ shipment_offer.destination.state }}
                        </p>
                      </template>
                      <p class="subtitle is-6">
                        <span :class="[(sort === 'miles') ? 'has-background-grey-lighter' : '']">{{ shipment_offer.miles | formatMiles }}</span>
                        for
                        <span :class="[(sort === 'price') ? 'has-background-grey-lighter' : '']">{{ shipment_offer.offer | formatCurrency }}</span>
                      </p>

                      <div :class="['box', (sort === 'pickupDate') ? 'has-background-grey-lighter' : '']">
                        <p class="title is-5">Pickup</p>
                        <p class="subtitle is-7">
                          Between {{ shipment_offer.origin.pickup.start | formatLocalTime }}
                          &amp; {{ shipment_offer.origin.pickup.end | formatLocalTime }}
                        </p>
                      </div>

                      <div :class="['box', (sort === 'dropoffDate') ? 'has-background-grey-lighter' : '']">
                        <p class="title is-5">Dropoff</p>
                        <p class="subtitle is-7">
                          Between {{ shipment_offer.destination.dropoff.start | formatLocalTime }}
                          &amp; {{ shipment_offer.destination.dropoff.end | formatLocalTime }}
                        </p>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </template>

          <div class="buttons is-centered">
            <a v-on:click="fetchMore" class="button is-primary is-large is-centered">Show More</a>
          </div>
        </div>
      </div>
    </template>

  </div>

</template>

<script>
import axios from "axios";
import extFormatCurrency from "format-currency";
import JQuery from "jquery";
import moment from "moment";
let $ = JQuery;

export default {
  name: "ShipmentOffers",
  data() {
    return {
      shipment_offers: [],
      loading: true,
      sort: "pickupDate",
      order: "desc",
      offset: 0,
      limit: 24,
      available_sorts: [
        "pickupDate",
        "dropoffDate",
        "price",
        "origin",
        "destination",
        "miles"
      ],
      axiosInstance: axios.create({
        baseURL: "https://convoy-frontend-homework-api.herokuapp.com",
        headers: {
          Accept: "application/json"
        }
      })
    };
  },
  methods: {
    fetchOffers: function() {
      var vm = this;
      vm.loading = false;

      var callback = function() {};

      var fetchHandler = function() {
        vm.loading = false;
        window.setTimeout(callback, 0);
      };

      vm.axiosInstance
        .get(
          "/offers?sort=" +
            vm.sort +
            "&order=" +
            vm.order +
            "&offset=" +
            vm.offset +
            "&limit=" +
            vm.limit
        )
        .then(function(response) {
          if (vm.offset === 0) {
            vm.shipment_offers = response.data;
          } else {
            vm.shipment_offers.push.apply(vm.shipment_offers, response.data);
          }
          fetchHandler();
        })
        .catch(function(error) {
          vm.shipment_offers = [];
          fetchHandler();
        });
    },
    fetchMore: function() {
      this.offset++;
      this.fetchOffers();
    },
    changeSort: function(newSort) {
      this.sort = newSort;
      this.loading = true;
      this.fetchOffers();
    },
    initializeSortDropdown: function() {
      var vm = this;

      $(".dropdown").on("click", function(e) {
        e.preventDefault();
        $(this).toggleClass("is-active");
      });
      $("body").on("click", function(e) {
        if ($(e.target).parents(".dropdown").length == 0) {
          $(".dropdown").removeClass("is-active");
        }
      });
      $(".dropdown a").on("click", function(e) {
        vm.changeSort($(this).data("new-sort"));
      });
    }
  },
  filters: {
    humanizeSort: function(value) {
      if (!value) return "";
      switch (value) {
        case "pickupDate":
          return "Pick-Up Date";
        case "dropoffDate":
          return "Drop-Off Date";
        default:
          return value.charAt(0).toUpperCase() + value.slice(1);
      }
    },
    formatMiles(value) {
      value = value.toString();
      var pattern = /(-?\d+)(\d{3})/;
      while (pattern.test(value)) value = value.replace(pattern, "$1,$2");
      return value + "mi";
    },
    formatCurrency(value) {
      return "$" + extFormatCurrency(value);
    },
    formatLocalTime(dateTime) {
      return moment(dateTime).format("MMMM Do YYYY, h:mm:ss a");
    }
  },
  mounted() {
    this.fetchOffers();
    this.initializeSortDropdown();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
[v-cloak] {
  display: none;
}
.section .dropdown {
  vertical-align: middle;
}
.buttons {
  width: 100%;
}
</style>
