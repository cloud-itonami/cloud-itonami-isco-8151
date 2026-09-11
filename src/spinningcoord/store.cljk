(ns spinningcoord.store
  "SSoT for the ISCO-08 8151 fibre preparing, spinning and winding
  machine operators mill scheduling/logistics coordination actor
  (itonami actor pattern, ADR-2607121000 / CLAUDE.md Actors section;
  README's 'Robotics premise' — a mill scheduling/logistics
  coordination robot performs crew scheduling, production-run/
  inventory/progress-record logging and raw-fibre/yarn-stock
  supply-order coordination for a fibre-preparing/spinning/winding
  crew under this advisor/governor pair, which never dispatches
  hardware itself, never operates spinning/winding equipment itself,
  and never finalizes a machine-operation-execution decision or a
  mill-safety-clearance decision, and never overrides a mill safety
  officer's judgment — those remain the mill safety officer's
  exclusive judgment). Modeled closely on cloud-itonami-isco-8122's
  platingcoord.store.

  Domain:

    spinner  — a registered fibre preparing/spinning/winding machine
               operator crew member (:spinner-id, :name)
    mill     — a registered spinning mill facility/line {:mill-id :name
               :max-supply-cost number}. `:max-supply-cost` is an
               informational registered ceiling used only to decide
               whether a `:coordinate-supply-order` proposal escalates
               to human sign-off (the governor never blocks a
               within-threshold order outright; it only decides
               commit vs. escalate).
    record   — a committed operating record (a logged production-run/
               inventory/progress entry, a scheduled crew/shift
               operation, a flagged safety concern, or a coordinated
               raw-fibre/yarn-stock supply order) — written ONLY via
               commit-record!. This actor coordinates mill
               scheduling/logistics ONLY — a `record` is a
               coordination artifact, never a machine-operation-
               execution act, never a mill-safety-clearance decision,
               and never a mill safety officer's-judgment override.
    ledger   — append-only audit trail, commit or hold.")

(defprotocol Store
  (spinner [s spinner-id])
  (mill [s mill-id])
  (records-of [s spinner-id])
  (ledger [s])
  (register-spinner! [s spinner])
  (register-mill! [s mill])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (spinner [_ spinner-id] (get-in @a [:spinners spinner-id]))
  (mill [_ mill-id] (get-in @a [:mills mill-id]))
  (records-of [_ spinner-id] (filter #(= spinner-id (:spinner-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-spinner! [s p]
    (swap! a assoc-in [:spinners (:spinner-id p)] p) s)
  (register-mill! [s f]
    (swap! a assoc-in [:mills (:mill-id f)] f) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:spinners {} :mills {} :records [] :ledger []}
                                    seed)))))
