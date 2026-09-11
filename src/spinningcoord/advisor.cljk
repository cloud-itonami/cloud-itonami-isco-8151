(ns spinningcoord.advisor
  "Fibre Preparing, Spinning and Winding Mill Scheduling Coordination
  Advisor — proposing a mill scheduling/logistics coordination
  operation (log a work record, schedule a crew operation, flag a
  safety concern, coordinate a raw-fibre/yarn-stock supply order) from
  a crew roster, mill registration and safety-reporting policy.
  Swappable mock/llm; the advisor ONLY proposes —
  `spinningcoord.governor` independently gates every proposal and
  always escalates safety concerns and above-threshold supply orders.
  The advisor never proposes to directly finalize a
  machine-operation-execution decision (e.g. deciding to proceed with
  a specific spinning or winding run), or a mill-safety-clearance
  decision (e.g. declaring the mill or a spinning line safety
  cleared), and never proposes to override a mill safety officer's
  judgment — those stay permanently out of this actor's scope.
  Modeled closely on cloud-itonami-isco-8122's platingcoord.advisor.

  A proposal: {:op :log-work-record|:schedule-crew-operation|
               :flag-safety-concern|:coordinate-supply-order
               :effect :propose :spinner-id str :mill-id str
               :cost number :hazard-type kw :task str :stake kw
               :confidence n :rationale str}"
  (:require [clojure.edn :as edn]))

(defprotocol Advisor
  (-advise [advisor store request] "request -> proposal map"))

(defn- rationale-for [op spinner-id mill-id hazard-type]
  (case op
    :log-work-record
    (str "logged work record for spinner " spinner-id " at mill " mill-id)

    :schedule-crew-operation
    (str "scheduled crew operation for spinning task at mill " mill-id)

    :flag-safety-concern
    (str "flagged " (name (or hazard-type :hazard)) " concern for spinner "
         spinner-id " at mill " mill-id " — routed for mill safety officer review")

    :coordinate-supply-order
    (str "coordinated supply order for spinner " spinner-id " at mill " mill-id)

    (str "proposed " (name op) " for spinner " spinner-id " at mill " mill-id)))

(defn- infer [_store {:keys [op stake spinner-id mill-id cost hazard-type task]}]
  {:op op
   :effect :propose
   :spinner-id spinner-id
   :mill-id mill-id
   :cost cost
   :hazard-type hazard-type
   :task task
   :stake (or stake :low)
   :confidence (case (or stake :low) :high 0.7 :medium 0.85 :low 0.95)
   :rationale (rationale-for op spinner-id mill-id hazard-type)})

(defn mock-advisor []
  (reify Advisor
    (-advise [_ store request] (infer store request))))

(def ^:private system-prompt
  "You are a fibre preparing, spinning and winding machine operators
   mill scheduling/logistics coordination advisor. Given a request,
   propose an :op (one of :log-work-record, :schedule-crew-operation,
   :flag-safety-concern, :coordinate-supply-order), the :spinner-id,
   :mill-id, and any :cost/:hazard-type/:task fields, an honest
   :confidence and a :stake. Never propose an op outside this closed
   list, and never propose to directly finalize a machine-operation-
   execution decision (e.g. deciding to proceed with a specific
   spinning or winding run), or a mill-safety-clearance decision (e.g.
   declaring the mill or a spinning line safety cleared), or to
   override a mill safety officer's judgment — those are always out
   of this actor's scope; it coordinates mill scheduling/logistics
   only and never operates spinning/winding equipment or clears the
   mill for operation itself. Safety concerns always require human
   sign-off regardless of confidence.")

(defn- parse-proposal [content]
  (try
    (let [p (edn/read-string content)]
      (if (map? p)
        (assoc p :effect :propose)
        {:op :unknown :effect :propose :confidence 0.0 :stake :high
         :rationale "unparseable LLM response"}))
    (catch #?(:clj Exception :cljs js/Error) _
      {:op :unknown :effect :propose :confidence 0.0 :stake :high
       :rationale "LLM response parse failure"})))

(defn llm-advisor
  [chat-model model-generate-fn gen-opts]
  (reify Advisor
    (-advise [_ _store request]
      (let [msgs [{:role :system :content system-prompt}
                  {:role :user :content (str "operation request: " (pr-str request))}]
            resp (model-generate-fn chat-model msgs gen-opts)]
        (parse-proposal (:content resp))))))
