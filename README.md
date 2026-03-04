# ============================================================
# ANALYSEUR D'INSCRIPTIONS, DE NOTES ET DE COHERENCE PEDAGOGIQUE
# ============================================================

donnees = [
    ("Sara", "Math", 12, "G1"),
    ("Sara", "Info", 14, "G1"),
    ("Ahmed", "Math", 9, "G2"),
    ("Adam", "Chimie", 18, "G1"),
    ("Sara", "Math", 11, "G1"),
    ("Bouchra", "Info", "abc", "G2"),
    ("", "Math", 10, "G1"),
    ("Yassine", "Info", 22, "G2"),
    ("Ahmed", "Info", 13, "G2"),
    ("Adam", "Math", None, "G1"),
    ("Sara", "Chimie", 16, "G1"),
    ("Adam", "Info", 7, "G1"),
    ("Ahmed", "Math", 9, "G2"),
    ("Hana", "Physique", 15, "G3"),
    ("Hana", "Math", 8, "G3"),
]

# ============================================================
# PARTIE 1 : NETTOYAGE ET VALIDATION
# ============================================================

def valider(enregistrement):
    nom, matiere, note, groupe = enregistrement

    if not nom or not nom.strip():
        return (False, "nom vide")
    if not matiere or not matiere.strip():
        return (False, "matiere vide")
    if not groupe or not groupe.strip():
        return (False, "groupe vide")
    try:
        note_num = float(note)
    except (TypeError, ValueError):
        return (False, "note non numerique")
    if note_num < 0 or note_num > 20:
        return (False, "note hors intervalle [0,20]")

    return (True, "")


valides = []
erreurs = []
doublons_exact = set()
vus = set()

for enreg in donnees:
    if enreg in vus:
        doublons_exact.add(enreg)
    else:
        vus.add(enreg)

    est_valide, raison = valider(enreg)
    if est_valide:
        nom, matiere, note, groupe = enreg
        valides.append((nom, matiere, float(note), groupe))
    else:
        erreurs.append({"ligne": enreg, "raison": raison})

print("=" * 50)
print("PARTIE 1 : NETTOYAGE")
print("=" * 50)
print(f"Valides   : {len(valides)}")
print(f"Erreurs   : {len(erreurs)}")
for e in erreurs:
    print(f"  -> {e['ligne']} | Raison : {e['raison']}")
print(f"Doublons  : {doublons_exact}")


# ============================================================
# PARTIE 2 : STRUCTURATION
# ============================================================

# Matières distinctes (set = pas de doublons)
matieres_distinctes = set()
for nom, matiere, note, groupe in valides:
    matieres_distinctes.add(matiere)

# Notes par étudiant par matière : { nom: { matiere: [notes] } }
notes_par_etudiant = {}
for nom, matiere, note, groupe in valides:
    if nom not in notes_par_etudiant:
        notes_par_etudiant[nom] = {}
    if matiere not in notes_par_etudiant[nom]:
        notes_par_etudiant[nom][matiere] = []
    notes_par_etudiant[nom][matiere].append(note)

# Étudiants par groupe : { groupe: set(noms) }
etudiants_par_groupe = {}
for nom, matiere, note, groupe in valides:
    if groupe not in etudiants_par_groupe:
        etudiants_par_groupe[groupe] = set()
    etudiants_par_groupe[groupe].add(nom)

print("\n" + "=" * 50)
print("PARTIE 2 : STRUCTURATION")
print("=" * 50)
print(f"Matières distinctes : {matieres_distinctes}")
print(f"Notes par étudiant  : {notes_par_etudiant}")
print(f"Groupes             : {etudiants_par_groupe}")


# ============================================================
# PARTIE 3 : CALCULS ET STATISTIQUES
# ============================================================

def somme_recursive(lst):
    if len(lst) == 0:
        return 0
    return lst[0] + somme_recursive(lst[1:])

def moyenne(lst):
    if len(lst) == 0:
        return 0
    return somme_recursive(lst) / len(lst)

print("\n" + "=" * 50)
print("PARTIE 3 : STATISTIQUES")
print("=" * 50)

for etudiant, matieres in notes_par_etudiant.items():
    toutes_notes = []
    print(f"\n{etudiant} :")
    for mat, notes in matieres.items():
        moy_mat = moyenne(notes)
        print(f"  {mat} -> notes : {notes} | moyenne : {moy_mat:.2f}")
        toutes_notes.extend(notes)
    moy_generale = moyenne(toutes_notes)
    print(f"  => Moyenne générale : {moy_generale:.2f}")


# ============================================================
# PARTIE 4 : ANALYSE AVANCEE ET DETECTION D'ANOMALIES
# ============================================================

alertes = {
    "notes_multiples": [],
    "profil_incomplet": [],
    "groupe_moyenne_faible": [],
    "ecart_important": []
}

SEUIL_GROUPE = 10
SEUIL_ECART  = 8

# 1. Étudiants avec plus d'une note par matière
for etudiant, matieres in notes_par_etudiant.items():
    for mat, notes in matieres.items():
        if len(notes) > 1:
            alertes["notes_multiples"].append({
                "etudiant": etudiant,
                "matiere": mat,
                "notes": notes
            })

# 2. Profil incomplet (n'a pas toutes les matières)
for etudiant, matieres in notes_par_etudiant.items():
    matieres_etud = set(matieres.keys())
    if matieres_etud != matieres_distinctes:
        manquantes = matieres_distinctes - matieres_etud
        alertes["profil_incomplet"].append({
            "etudiant": etudiant,
            "matieres_manquantes": manquantes
        })

# 3. Groupes avec moyenne générale faible
for groupe, etudiants in etudiants_par_groupe.items():
    toutes_notes_groupe = []
    for etudiant in etudiants:
        for notes in notes_par_etudiant[etudiant].values():
            toutes_notes_groupe.extend(notes)
    moy_groupe = moyenne(toutes_notes_groupe)
    if moy_groupe < SEUIL_GROUPE:
        alertes["groupe_moyenne_faible"].append({
            "groupe": groupe,
            "moyenne": round(moy_groupe, 2)
        })

# 4. Écart important entre note min et max
for etudiant, matieres in notes_par_etudiant.items():
    toutes_notes = []
    for notes in matieres.values():
        toutes_notes.extend(notes)
    if len(toutes_notes) >= 2:
        ecart = max(toutes_notes) - min(toutes_notes)
        if ecart > SEUIL_ECART:
            alertes["ecart_important"].append({
                "etudiant": etudiant,
                "ecart": ecart,
                "min": min(toutes_notes),
                "max": max(toutes_notes)
            })

print("\n" + "=" * 50)
print("PARTIE 4 : ANOMALIES DETECTEES")
print("=" * 50)

print("\n[1] Notes multiples par matière :")
for a in alertes["notes_multiples"]:
    print(f"  {a['etudiant']} | {a['matiere']} : {a['notes']}")

print("\n[2] Profils incomplets :")
for a in alertes["profil_incomplet"]:
    print(f"  {a['etudiant']} manque : {a['matieres_manquantes']}")

print("\n[3] Groupes à moyenne faible (seuil={}) :".format(SEUIL_GROUPE))
for a in alertes["groupe_moyenne_faible"]:
    print(f"  {a['groupe']} -> moyenne : {a['moyenne']}")

print("\n[4] Étudiants avec grand écart (seuil={}) :".format(SEUIL_ECART))
for a in alertes["ecart_important"]:
    print(f"  {a['etudiant']} | min={a['min']} max={a['max']} écart={a['ecart']}")
