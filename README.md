# 🤖 Παράδειγμα Chatbot με Rasa – Μάθημα MSc - Διαλογικοί Πράκτορες

Αυτό το αποθετήριο περιέχει ένα έτοιμο chatbot βασισμένο στο **Rasa Open Source**, κατάλληλο για πειραματισμό και εκμάθηση βασικών εννοιών της επεξεργασίας φυσικής γλώσσας (NLP) και της ανάπτυξης διαλογικών πρακτόρων (chatbots).

## ✅ Τι είναι το Rasa;

Το **Rasa** είναι ένα framework για τη δημιουργία έξυπνων chatbot με Python, βασισμένο στην αναγνώριση προθέσεων (intents), την κατηγορίαση κειμένου κ.ά. και τη διαχείριση διαλόγου.

---

## ⚙️ Πώς ξεκινάμε;

1. **Δημιούργησε ένα Codespace**
   Ξεκινώντας από αυτό το repository, μπορείς να επιλέξεις το κουμπί "Use this template" για να φτιάξεις ένα νέο repository με βάση αυτό, ή να το ανοίξεις κατευθείαν μέσω Codespaces. Αν δημιουργήσεις πρώτα ένα νέο repo με βάση αυτό, τότε στη συνέχεια μπορείς στο νέο σου repo, να επιλέξεις το πράσινο κουμπί **“Code”** → **“Create codespace on main”** για να ανοίξεις το project στο Codespaces.

3. **Εκπαίδευσε το μοντέλο**
   στο παρόν παράδειγμα έχει ήδη γίνει train για την υπάρχουσα δομή του bot, αλλά έπειτα από κάθε νέα προσθήκη/αλλαγή που κάνεις, θα πρέπει να ξανατρέξεις το:
   ```bash
   rasa train
   ```
   για να ενσωματωθούν οι αλλαγές σου.

5. **Μίλησε με το bot**
   ```bash
   rasa shell
   ```

---

## 🗂️ Δομή του Project

Η δομή που έχει το project είναι και η δομή που έχει κάθε rasa project που δημιουργείται μέσω της εντολής rasa init, και τα βασικά του αρχεία και φάκελοι είναι τα ακόλουθα:

- `data/nlu.yml`: Παραδείγματα προτάσεων και προθέσεων του χρήστη (π.χ. χαιρετισμοί, ερωτήματα)
- `domain.yml`: Ορισμός των απαντήσεων και των δράσεων του bot
- `data/stories.yml`: Σενάρια διαλόγων
- `actions.py`: Προγραμματισμός προσαρμοσμένων ενέργειων
- `config.yml`: Ρυθμίσεις για το NLP pipeline του Rasa

---

## 🧚 Παράδειγμα – δημιουργία intent

1. Πρόσθεσε μια νέα πρόθεση (intent) στο `nlu.yml`  
   ```yaml
   - intent: ask_course_info
     examples: |
       - What courses are included in the schedule?
       - What is included in the MSc?
   ```

2. Πρόσθεσε μια απάντηση στο `domain.yml`  
   ```yaml
   responses:
     utter_course_info:
       - text: "The schedule includes both theoretical and practical courses regarding Artificial Intelligence and its applications in eduational settings."
   ```

3. Πρόσθεσε ένα σενάριο στο `stories.yml`  
   ```yaml
   - story: Course information
     steps:
       - intent: ask_course_info
       - action: utter_course_info
   ```

4. Εκπαίδευσε ξανά:
   ```bash
   rasa train
   ```

5. Δοκίμασέ το με `rasa shell` και γράψε μια από τις φράσεις που πρόσθεσες!

---

## 🧚 Παράδειγμα – προσθήκη entity/slot

1. Πρόσθεσε ένα νέο entity και το αντίστοιχο slot στο `domain.yml`
   ```yaml
   entities:
     - mood

   slots:
     mood:
       type: text
       influence_conversation: true
       mappings:
       - type: from_entity
         entity: mood
   ```

2. Πρόσθεσε (ή ενημέρωσε) τα training examples στο `nlu.yml`, σημειώνοντας τα entities με `[value](entity_name)`
   ```yaml
   - intent: mood_great
     examples: |
       - I'm feeling [happy](mood)
       - I'm [great](mood)
       - Feeling really [good](mood)

   - intent: mood_unhappy
     examples: |
       - [sad](mood)
       - [unhappy](mood)
       - [not good](mood)
   ```

3. Πρόσθεσε μια **νέα** απάντηση του bot που χρησιμοποιεί το slot, στο `domain.yml`
   ```yaml
   responses:
     utter_mood:
     - text: "I see you're feeling {mood}"
   ```
   Πρόσεξε: δημιουργούμε μια νέα απάντηση `utter_mood` αντί να αλλάζουμε κάποια υπάρχουσα.

4. Ενημέρωσε το `stories.yml` ώστε να χρησιμοποιεί τη νέα απάντηση
   ```yaml
   - story: sad path 1
     steps:
     - intent: greet
     - action: utter_greet
     - intent: mood_unhappy
     - action: utter_mood
     - action: utter_cheer_up
     - action: utter_did_that_help
     - intent: affirm
     - action: utter_happy
   ```

5. Εκπαίδευσε ξανά:
   ```bash
   rasa train
   ```

6. Δοκίμασέ το με `rasa shell`!

---

## 🧚 Παράδειγμα – προσθήκη rule

1. Πρόσθεσε ένα νέο rule στο `rules.yml`
   ```yaml
   rules:
   - rule: fallback for unclear messages
     steps:
     - intent: nlu_fallback
     - action: utter_default
   ```

2. Πρόσθεσε ένα νέο intent και ένα γενικό response στο `domain.yml`
   ```yaml
   intents:
     - nlu_fallback

   responses:
     utter_default:
     - text: "Sorry, I didn't understand that. Could you rephrase?"
   ```

3. Εκπαίδευσε ξανά:
   ```bash
   rasa train
   ```

4. Δοκίμασέ το με `rasa shell`!

---

## 🧚 Παράδειγμα – προσθήκη form

Τα forms χρησιμοποιούνται όταν το bot χρειάζεται να συλλέξει πολλές πληροφορίες από τον χρήστη, ρωτώντας τον μία-μία. Το Rasa διαχειρίζεται αυτόματα τη ροή: ρωτάει κάθε πεδίο με τη σειρά, και αν ο χρήστης δώσει κάποια πληροφορία νωρίτερα, το bot δεν τη ζητάει ξανά.

Σε αυτό το παράδειγμα θα φτιάξουμε μια φόρμα feedback, όπου ο χρήστης δίνει το όνομά του και ένα σχόλιο.

1. Πρόσθεσε τα slots και τη φόρμα στο `domain.yml`
   ```yaml
   slots:
     name:
       type: text
       influence_conversation: false
       mappings:
       - type: from_text
         conditions:
         - active_loop: feedback_form
           requested_slot: name
     feedback_message:
       type: text
       influence_conversation: false
       mappings:
       - type: from_text
         conditions:
         - active_loop: feedback_form
           requested_slot: feedback_message

   forms:
     feedback_form:
       required_slots:
       - name
       - feedback_message
   ```
   Και τα δύο slots χρησιμοποιούν `from_text`, που σημαίνει ότι καταγράφουν ολόκληρο το μήνυμα του χρήστη ως τιμή — αλλά μόνο όταν η φόρμα είναι ενεργή και ζητάει το συγκεκριμένο slot (χάρη στο `conditions`).

2. Πρόσθεσε τα responses που χρησιμοποιεί η φόρμα για να ρωτάει κάθε slot, στο `domain.yml`
   ```yaml
   responses:
     utter_ask_name:
     - text: "What is your name?"
     utter_ask_feedback_message:
     - text: "Thanks {name}! Please write your feedback:"
     utter_feedback_thanks:
     - text: "Thank you {name}, your feedback has been received!"
   ```
   Πρόσεξε: τα responses `utter_ask_<slot_name>` πρέπει να ταιριάζουν ακριβώς με τα ονόματα των slots.

3. Πρόσθεσε ένα νέο intent στο `nlu.yml`
   ```yaml
   - intent: give_feedback
     examples: |
       - I want to give feedback
       - I'd like to leave a comment
       - feedback
       - I have some feedback
       - can I leave feedback?
   ```

4. Πρόσθεσε τα rules που ενεργοποιούν και ολοκληρώνουν τη φόρμα, στο `rules.yml`
   ```yaml
   - rule: Activate feedback form
     steps:
     - intent: give_feedback
     - action: feedback_form
     - active_loop: feedback_form

   - rule: Submit feedback form
     condition:
     - active_loop: feedback_form
     steps:
     - action: feedback_form
     - active_loop: null
     - action: utter_feedback_thanks
   ```
   Πρόσεξε: τα forms ενεργοποιούνται και ολοκληρώνονται μέσω rules (όχι stories), γιατί η ροή τους είναι πάντα η ίδια.

5. Μην ξεχάσεις να προσθέσεις το intent `give_feedback` στη λίστα intents του `domain.yml`
   ```yaml
   intents:
     - give_feedback
   ```

6. Εκπαίδευσε ξανά:
   ```bash
   rasa train
   ```

7. Δοκίμασέ το με `rasa shell` — γράψε "I want to give feedback" και ακολούθησε τις ερωτήσεις!

---

## 📚 Χρήσιμοι Σύνδεσμοι

- [Rasa Documentation (EN)](https://legacy-docs-oss.rasa.com/docs/rasa/)
- [Rasa YouTube Tutorials (EN)](https://www.youtube.com/watch?v=Ap62n_YAVZ8&list=PL75e0qA87dlEjGAc9j9v3a5h1mxI2Z9fi)

---

## ℹ️ Σημειώσεις

- Το chatbot αυτό βασίζεται στο επίσημο Rasa demo (`rasa init`).
- Δεν χρειάζεσαι εμπειρία στον προγραμματισμό για να πειραματιστείς με τις βασικές λειτουργίες.
- Όλο το περιβάλλον είναι διαθέσιμο μέσω του browser με GitHub Codespaces.

---

Have fun! 🎓
