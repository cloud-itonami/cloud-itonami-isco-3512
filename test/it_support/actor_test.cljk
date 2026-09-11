(ns it-support.actor-test
  (:require [clojure.test :refer [deftest is testing]]
            [it-support.actor :as actor]
            [it-support.store :as store]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-ticket! st {:ticket-id "ticket-1" :name "Laptop won't boot"})
    st))

(deftest commits-a-clean-low-risk-request
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:ticket-id "ticket-1" :op :diagnose :stake :low}
        result (actor/run-request! graph request {} "thread-1")]
    (is (= :done (:status result)))
    (is (some? (get-in result [:state :record])))
    (is (= 1 (count (store/records-of st "ticket-1"))))))

(deftest holds-on-unregistered-ticket-without-committing
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:ticket-id "no-such-ticket" :op :diagnose :stake :low}
        result (actor/run-request! graph request {} "thread-2")]
    (is (= :done (:status result)))
    (is (nil? (get-in result [:state :record])))
    (is (empty? (store/records-of st "no-such-ticket")))
    (is (= :hold (:disposition (:state result))))))

(deftest interrupts-then-commits-on-human-approval
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        ;; credential reset always escalates (governor invariant)
        request {:ticket-id "ticket-1" :op :reset-credentials :stake :high}
        interrupted (actor/run-request! graph request {} "thread-3")]
    (is (= :interrupted (:status interrupted)))
    (is (empty? (store/records-of st "ticket-1")))
    (let [resumed (actor/approve! graph "thread-3")]
      (is (= :done (:status resumed)))
      (is (some? (get-in resumed [:state :record])))
      (is (= 1 (count (store/records-of st "ticket-1")))))))
