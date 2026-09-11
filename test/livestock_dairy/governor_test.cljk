(ns livestock-dairy.governor-test
  (:require [clojure.test :refer [deftest is testing]]
            [livestock-dairy.store :as store]
            [livestock-dairy.governor :as governor]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-herd! st {:herd-id "herd-1" :name "North Pasture Herd"})
    st))

(deftest ok-on-clean-feed
  (let [st (fresh-store)
        proposal {:op :feed :effect :propose :confidence 0.9 :stake :low}
        v (governor/check {:herd-id "herd-1"} {} proposal st)]
    (is (:ok? v))
    (is (not (:hard? v)))
    (is (not (:escalate? v)))))

(deftest hard-on-unregistered-herd
  (let [st (fresh-store)
        proposal {:op :feed :effect :propose :confidence 0.9 :stake :low}
        v (governor/check {:herd-id "no-such-herd"} {} proposal st)]
    (is (:hard? v))
    (is (some #(= :no-herd (:rule %)) (:violations v)))))

(deftest hard-on-no-actuation-violation
  (let [st (fresh-store)
        proposal {:op :feed :effect :direct-write :confidence 0.9 :stake :low}
        v (governor/check {:herd-id "herd-1"} {} proposal st)]
    (is (:hard? v))
    (is (some #(= :no-actuation (:rule %)) (:violations v)))))

(deftest escalates-on-veterinary-medication-administration
  (let [st (fresh-store)
        proposal {:op :administer-veterinary-medication :effect :propose :confidence 0.9 :stake :high}
        v (governor/check {:herd-id "herd-1"} {} proposal st)]
    (is (:escalate? v))
    (is (not (:hard? v)))))

(deftest escalates-on-operate-near-large-livestock
  (let [st (fresh-store)
        proposal {:op :operate-near-large-livestock :effect :propose :confidence 0.9 :stake :high}
        v (governor/check {:herd-id "herd-1"} {} proposal st)]
    (is (:escalate? v))
    (is (not (:hard? v)))))

(deftest escalates-on-low-confidence
  (let [st (fresh-store)
        proposal {:op :feed :effect :propose :confidence 0.2 :stake :low}
        v (governor/check {:herd-id "herd-1"} {} proposal st)]
    (is (:escalate? v))
    (is (not (:hard? v)))))

(deftest store-records-and-ledger-append-only
  (let [st (fresh-store)]
    (store/commit-record! st {:herd-id "herd-1" :op :monitor-health})
    (store/append-ledger! st {:disposition :commit})
    (is (= 1 (count (store/records-of st "herd-1"))))
    (is (= 1 (count (store/ledger st))))))
