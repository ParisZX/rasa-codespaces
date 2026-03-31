# 🧚 Παράδειγμα – δημιουργία Custom Action

Τα **Custom Actions** σου επιτρέπουν να εκτελέσεις κώδικα Python όταν το bot φτάσει σε ένα συγκεκριμένο σημείο του διαλόγου. Μπορείς π.χ. να καλέσεις ένα API, να κάνεις υπολογισμούς, να διαβάσεις/γράψεις σε βάση δεδομένων, ή να δημιουργήσεις δυναμικές απαντήσεις.

Τα custom actions ορίζονται στο αρχείο `actions/actions.py` και εκτελούνται από έναν ξεχωριστό server (Rasa SDK Action Server).

---

## Πώς λειτουργεί

1. Ο χρήστης στέλνει ένα μήνυμα → το Rasa αναγνωρίζει intent
2. Βάσει stories/rules, το Rasa αποφασίζει ότι πρέπει να εκτελεστεί ένα custom action
3. Το Rasa στέλνει αίτημα στον **Action Server** (port 5055)
4. Ο Action Server εκτελεί τον Python κώδικα και επιστρέφει την απάντηση

---

## Βήμα 1 – Γράψε το action στο `actions/actions.py`

Σε αυτό το παράδειγμα φτιάχνουμε ένα action που επιστρέφει την τρέχουσα ημερομηνία και ώρα:

```python
from typing import Any, Text, Dict, List
from datetime import datetime

from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher


class ActionTellTime(Action):

    def name(self) -> Text:
        return "action_tell_time"

    def run(self, dispatcher: CollectingDispatcher,
            tracker: Tracker,
            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:

        now = datetime.now().strftime("%H:%M, %d/%m/%Y")
        dispatcher.utter_message(text=f"The current time is {now}.")

        return []
```

Κάθε action πρέπει να:
- Κληρονομεί την κλάση `Action`
- Υλοποιεί τη μέθοδο `name()` που επιστρέφει ένα μοναδικό όνομα (ξεκινάει πάντα με `action_`)
- Υλοποιεί τη μέθοδο `run()` που περιέχει τον κώδικα που θέλεις να εκτελεστεί

Η μέθοδος `run()` δέχεται τρία arguments:
- **`dispatcher`** – χρησιμοποιείται για να στείλεις μήνυμα πίσω στον χρήστη (`dispatcher.utter_message(...)`)
- **`tracker`** – περιέχει το ιστορικό της συνομιλίας, τα slots, το τελευταίο intent κ.λπ. (π.χ. `tracker.get_slot("name")`)
- **`domain`** – το domain του bot (intents, responses, slots κ.λπ.)

---

## Βήμα 2 – Δήλωσε το action στο `domain.yml`

Πρόσθεσε το action στη λίστα `actions` και ένα νέο intent με τα αντίστοιχα NLU examples:

```yaml
intents:
  - ask_time

actions:
  - action_tell_time

responses:
  utter_ask_time_fallback:
  - text: "Sorry, I couldn't get the time right now."
```

---

## Βήμα 3 – Πρόσθεσε training data στο `data/nlu.yml`

```yaml
- intent: ask_time
  examples: |
    - What time is it?
    - Tell me the time
    - What's the current time?
    - Do you know what time it is?
    - What hour is it?
```

---

## Βήμα 4 – Πρόσθεσε rule στο `data/rules.yml`

Τα custom actions συνήθως ενεργοποιούνται μέσω rules:

```yaml
- rule: Tell the time
  steps:
  - intent: ask_time
  - action: action_tell_time
```

---

## Βήμα 5 – Ενεργοποίησε τον Action Server στο `endpoints.yml`

Ξεκομμεντάρισε (ή πρόσθεσε) τo `action_endpoint`:

```yaml
action_endpoint:
  url: "http://localhost:5055/webhook"
```

---

## Βήμα 6 – Εκπαίδευσε και τρέξε

Χρειάζονται **δύο terminals** — ένα για τον Action Server και ένα για το bot:

**Terminal 1** – Ξεκίνα τον Action Server:
```bash
rasa run actions
```

**Terminal 2** – Εκπαίδευσε και μίλα με το bot:
```bash
rasa train
rasa shell
```

Γράψε "What time is it?" και θα πάρεις την τρέχουσα ώρα!

---

## 💡 Συμβουλές

- Τα ονόματα των actions πρέπει **πάντα** να ξεκινούν με `action_`
- Αν θέλεις να χρησιμοποιήσεις εξωτερικές βιβλιοθήκες (π.χ. `requests`), εγκατάστησέ τες πρώτα με `pip install`
- Μπορείς να χρησιμοποιήσεις τον `tracker` για να διαβάσεις slots: `tracker.get_slot("slot_name")`
- Μπορείς να στείλεις εικόνες, κουμπιά ή custom JSON μέσω του `dispatcher`:
  ```python
  dispatcher.utter_message(
      text="Choose an option:",
      buttons=[
          {"title": "Option A", "payload": "/intent_a"},
          {"title": "Option B", "payload": "/intent_b"},
      ]
  )
  ```
- Μην ξεχνάς: κάθε φορά που αλλάζεις τον κώδικα στο `actions.py`, πρέπει να **κάνεις restart τον Action Server**

---

## 📚 Περισσότερα

- [Rasa Custom Actions Documentation](https://legacy-docs-oss.rasa.com/docs/rasa/custom-actions)
- [Rasa SDK Documentation](https://legacy-docs-oss.rasa.com/docs/action-server)
