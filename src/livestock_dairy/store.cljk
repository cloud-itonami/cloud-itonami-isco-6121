(ns livestock-dairy.store
  "SSoT for the ISCO-08 6121 independent livestock-and-dairy sole-
  proprietor actor. Store is a protocol injected into the
  `livestock-dairy.actor` StateGraph — `MemStore` is the default,
  deterministic, zero-dep backend; a Datomic/kotoba-server-backed
  implementation can be swapped in without touching the actor or
  governor (itonami actor pattern, per ADR-2607011000 / CLAUDE.md
  Actors section).

  Domain:

    herd     — a registered herd (:herd-id, :name)
    record   — a committed operating record under a herd (feed step,
               health-monitoring entry, veterinary-medication
               administration, operation near large livestock) —
               written ONLY via commit-record!, never mutated in
               place
    ledger   — an append-only audit trail of every proposal/verdict/
               disposition, regardless of outcome (commit or hold)")

(defprotocol Store
  (herd [s herd-id])
  (records-of [s herd-id])
  (ledger [s])
  (register-herd! [s herd])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (herd [_ herd-id] (get-in @a [:herds herd-id]))
  (records-of [_ herd-id] (filter #(= herd-id (:herd-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-herd! [s herd]
    (swap! a assoc-in [:herds (:herd-id herd)] herd) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:herds {} :records [] :ledger []} seed)))))
