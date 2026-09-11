(ns it-support.governor-test
  (:require [clojure.test :refer [deftest is testing]]
            [it-support.store :as store]
            [it-support.governor :as governor]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-ticket! st {:ticket-id "ticket-1" :name "Laptop won't boot"})
    st))

(deftest ok-on-clean-diagnose
  (let [st (fresh-store)
        proposal {:op :diagnose :effect :propose :confidence 0.9 :stake :low}
        v (governor/check {:ticket-id "ticket-1"} {} proposal st)]
    (is (:ok? v))
    (is (not (:hard? v)))
    (is (not (:escalate? v)))))

(deftest hard-on-unregistered-ticket
  (let [st (fresh-store)
        proposal {:op :diagnose :effect :propose :confidence 0.9 :stake :low}
        v (governor/check {:ticket-id "no-such-ticket"} {} proposal st)]
    (is (:hard? v))
    (is (some #(= :no-ticket (:rule %)) (:violations v)))))

(deftest hard-on-no-actuation-violation
  (let [st (fresh-store)
        proposal {:op :diagnose :effect :direct-write :confidence 0.9 :stake :low}
        v (governor/check {:ticket-id "ticket-1"} {} proposal st)]
    (is (:hard? v))
    (is (some #(= :no-actuation (:rule %)) (:violations v)))))

(deftest escalates-on-credential-reset
  (let [st (fresh-store)
        proposal {:op :reset-credentials :effect :propose :confidence 0.9 :stake :high}
        v (governor/check {:ticket-id "ticket-1"} {} proposal st)]
    (is (:escalate? v))
    (is (not (:hard? v)))))

(deftest escalates-on-access-permission-change
  (let [st (fresh-store)
        proposal {:op :change-access-permissions :effect :propose :confidence 0.9 :stake :high}
        v (governor/check {:ticket-id "ticket-1"} {} proposal st)]
    (is (:escalate? v))
    (is (not (:hard? v)))))

(deftest escalates-on-low-confidence
  (let [st (fresh-store)
        proposal {:op :diagnose :effect :propose :confidence 0.2 :stake :low}
        v (governor/check {:ticket-id "ticket-1"} {} proposal st)]
    (is (:escalate? v))
    (is (not (:hard? v)))))

(deftest store-records-and-ledger-append-only
  (let [st (fresh-store)]
    (store/commit-record! st {:ticket-id "ticket-1" :op :resolve})
    (store/append-ledger! st {:disposition :commit})
    (is (= 1 (count (store/records-of st "ticket-1"))))
    (is (= 1 (count (store/ledger st))))))
