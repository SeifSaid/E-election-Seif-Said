//Projet C 
//E-election
//Preparer par: Said Seifeddine
//II 1 groupe B

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

char s[10];
//Listes chainée utilisée
//Ces listes seront sauvegardées dans 3 fichiers binaires
struct Utilisateur* users_tete= NULL;
struct Utilisateur* condidat_tete= NULL;
struct Vote* vote_tete= NULL;

//Stuctures de données

struct Utilisateur {
    char nom_prenom[50];
    char mot_de_passe[50]; 
    char CIN[10];
    int est_admin;       
    struct Utilisateur* suiv;
};


struct Vote {
    struct Utilisateur Voter;
    struct Utilisateur Condidat;
    struct Vote* suiv;

};

struct Resultat {
    char nom_prenom[50];
    int votes;
};
//Fonctions
//files
struct Utilisateur* CreateOpenUsers() {
    FILE* file = fopen("users.bin", "ab+");
    struct Utilisateur* head = NULL;

    struct Utilisateur user;
    while (fread(&user, sizeof(struct Utilisateur), 1, file) == 1) {
        struct Utilisateur* newNode = (struct Utilisateur*)malloc(sizeof(struct Utilisateur));
        memcpy(newNode, &user, sizeof(struct Utilisateur));
        newNode->suiv = head;
        head = newNode;
    }
    fclose(file);

    return head;
}
struct Utilisateur* CreateOpenCondidats() {
    FILE* file = fopen("condidats.bin", "ab+");
    struct Utilisateur* head = NULL;

    struct Utilisateur user;
    while (fread(&user, sizeof(struct Utilisateur), 1, file) == 1) {
        struct Utilisateur* newNode = (struct Utilisateur*)malloc(sizeof(struct Utilisateur));
        memcpy(newNode, &user, sizeof(struct Utilisateur));
        newNode->suiv = head;
        head = newNode;
    }
    fclose(file);

    return head;
}
struct Vote* CreateOpenVotes() {
    FILE* file = fopen("votes.bin", "ab+");
    struct Vote* head = NULL;
    struct Vote vote;
    while (fread(&vote, sizeof(struct Vote), 1, file) == 1) {
        struct Vote* newNode = (struct Vote*)malloc(sizeof(struct Vote));

        memcpy(newNode, &vote, sizeof(struct Vote));

        newNode->suiv = head;
        head = newNode;
    }

    fclose(file);
    return head;
}

//fonction pour affichage / test

void showuser(const struct Utilisateur* user) {
    printf("Nom et prenom: %s\n", user->nom_prenom);
    printf("Mot de passe : %s\n", user->mot_de_passe);
    printf("CIN          : %s\n", user->CIN);
    printf("Est admin    : %d\n", user->est_admin);
}
//min et max
int min(int a, int b) {
    return (a < b) ? a : b;
};
int max(int a, int b) {
    return (a > b) ? a : b;
};
//fonction qui demande de saisir un nombre entre a et b
int Option_test(int a,int b){
    int n;
    do {
        printf("Saisir un nombre entre %d et %d: ",a,b);
        scanf("%d", &n);
        if (n < min(a,b) || n > max(a,b)) {
            printf("Entrée invalide!\n");
        }

    } while (n < min(a,b) || n > max(a,b));
    return n;
}
//fonction longeur de la liste chainée
int Len(struct Utilisateur* tete) {
    int count = 0;
    struct Utilisateur* current = tete;

    while (current != NULL) {
        count++;
        current = current->suiv;
    }

    return count;
}

int Lenn(struct Vote* tete) {
    int count = 0;
    struct Vote* current = tete;

    while (current != NULL) {
        count++;
        current = current->suiv;
    }

    return count;
}
//fouction pour ajout d'un utilisateur
struct Utilisateur* add_User(struct Utilisateur* tete, struct Utilisateur* new_user) {
    
    struct Utilisateur* nouveau = (struct Utilisateur*)malloc(sizeof(struct Utilisateur));
    *nouveau = *new_user;
    nouveau->suiv = NULL;
    if (tete == NULL) {
        return nouveau;
    }
    struct Utilisateur* courant = tete;
    while (courant->suiv != NULL) {
        courant = courant->suiv;
    }
    courant->suiv = nouveau;
    return tete;
}
//retourne un pointeur vers un utilisateur dont on sait CIN
struct Utilisateur* findUserByCIN(const char* inputCIN, struct Utilisateur* users_tete) {
    struct Utilisateur* current = users_tete;
    while (current != NULL) {
        if (strcmp(current->CIN, inputCIN) == 0) {
            return current;
        }
        current = current->suiv;
    }
    return NULL;
}
int Ajout_condidat(){
    char cin [10];
    do{
    printf("< Ajouter un condidat à l'élection >\n\n");
    printf("Saisir le numero de CIN du condidat: ");
    fgets(cin, sizeof(cin), stdin);
    cin[strcspn(cin, "\n")] = '\0';}
    while (findUserByCIN(cin,users_tete)==NULL);
    condidat_tete=add_User(condidat_tete,findUserByCIN(cin,users_tete));
    printf("Condidat ajouté avec succès \n\n");
    menu_admin();
    return 0;
}

int Change_condidat(){
    char cin [10];
    do{
    printf("< Changer les données d'un condidat >\n\n");
    printf("Saisir le numero de CIN du condidat: ");
    fgets(cin, sizeof(cin), stdin);
    cin[strcspn(cin, "\n")] = '\0';
    if (findUserByCIN(cin,users_tete) == NULL) {
            printf("Utilisateur non trouvé. Veuillez réessayer.\n");
        }
    }
    while (findUserByCIN(cin,users_tete)==NULL);
    condidat_tete=add_User(condidat_tete,findUserByCIN(cin,users_tete));
    printf("Saisir votre nom et prenom    : ");
    fgets(findUserByCIN(cin,users_tete)->nom_prenom, sizeof(findUserByCIN(cin,users_tete)->nom_prenom), stdin);
    findUserByCIN(cin,users_tete)->nom_prenom[strcspn(findUserByCIN(cin,users_tete)->nom_prenom, "\n")] = '\0';
    printf("Saisir le numero de votre CIN : ");
    fgets(findUserByCIN(cin,users_tete)->CIN, sizeof(findUserByCIN(cin,users_tete)->CIN), stdin);
    findUserByCIN(cin,users_tete)->CIN[strcspn(findUserByCIN(cin,users_tete)->CIN, "\n")] = '\0';
    printf("Choisir votre mot de passe    : ");
    fgets(findUserByCIN(cin,users_tete)->mot_de_passe, sizeof(findUserByCIN(cin,users_tete)->mot_de_passe), stdin);
    findUserByCIN(cin,users_tete)->mot_de_passe[strcspn(findUserByCIN(cin,users_tete)->mot_de_passe, "\n")] = '\0';
    printf("Données du condidat changées avec succès \n\n");
    menu_admin();
    return 0;
}

void Show_res(struct Vote* tete) {
    if (tete == NULL) {
        printf("Aucun vote enregistré.\n");
        return;
    }
    struct Vote* current = tete;
    int maxCandidates = 100; 
    struct Resultat resultats[maxCandidates];
    for (int i = 0; i < maxCandidates; i++) {
        resultats[i].votes = 0;
    }
    while (current != NULL) {
        const char* candidateNomPrenom = current->Condidat.nom_prenom;
        int candidateIndex = -1;
        for (int i = 0; i < maxCandidates; i++) {
            if (strcmp(resultats[i].nom_prenom, candidateNomPrenom) == 0) {
                candidateIndex = i;
                break;
            }
        }
        if (candidateIndex >= 0 && candidateIndex < maxCandidates) {
            resultats[candidateIndex].votes++;
        }
        current = current->suiv;
    }

    // Sort (descending )
    for (int i = 0; i < maxCandidates - 1; i++) {
        for (int j = i + 1; j < maxCandidates; j++) {
            if (resultats[i].votes < resultats[j].votes) {
                struct Resultat temp = resultats[i];
                resultats[i] = resultats[j];
                resultats[j] = temp;
            }
        }
    }

    printf("\n < Résultats du vote > :\n");
    float totalVotes = Lenn(vote_tete) ;
    for (int i = 0; i < maxCandidates && resultats[i].votes > 0; i++) {
        float percentage = (resultats[i].votes / totalVotes) * 100.0;
        printf("%s : %d votes (%.2f%%)\n", resultats[i].nom_prenom, resultats[i].votes, percentage);
    }
    printf("\nLe gagnant de l'élection est : %s\n", resultats[0].nom_prenom);
}

void Show_Vote(struct Vote* vote_tete) {
    if (vote_tete == NULL) {
        printf("Pas de votes.\n");
        return;
    }

    struct Vote* current = vote_tete;
    while (current != NULL) {
        printf("Utilisateur: %s\n", current->Voter.nom_prenom);
        printf("Le condidat qu'il a voté : %s\n", current->Condidat.nom_prenom);
        printf("------------------------\n");
        current = current->suiv;
    }
}

int menu_admin(){
    printf("\n< Menu Administrateur >\n\n");
    printf("(1) Ajouter un condidat\n");
    printf("(2) Changer les données d'un condidat\n");
    printf("(3) Afficher les résultats de l'élection\n");
    printf("(4) Afficher les résultats de l'élection\n");
    printf("(0) Revenir au meun Principal\n");
    printf("\n");
    int login_input= Option_test(0,4);
    printf("*******\n");
    if(login_input==0) {
        Menu_principal();
    } else if(login_input==1) {
        Ajout_condidat();
        menu_admin();
    } else if(login_input==3) {
        Show_res(vote_tete);
        menu_admin();
    } else if(login_input==4) {
        Show_Vote(vote_tete);
        menu_admin();
    } else if(login_input==2) {
        Change_condidat();
        menu_admin();
    }
    return 0;
}


int Aff_condidats(struct Utilisateur* tete) {
    struct Utilisateur* current = tete;
    int index = 1;

    while (current != NULL) {
        printf("%d. %s\n", index, current->nom_prenom);
        current = current->suiv;
        index++;
    }
    return 0;
}

struct Vote* AjoutVote(struct Vote* tete, struct Utilisateur* voter, struct Utilisateur* condidat) {
    struct Vote* nouveauVote = (struct Vote*)malloc(sizeof(struct Vote));

    memcpy(&nouveauVote->Voter, voter, sizeof(struct Utilisateur));
    memcpy(&nouveauVote->Condidat, condidat, sizeof(struct Utilisateur));
    nouveauVote->suiv = tete;
    return nouveauVote;
}

struct Utilisateur* GetNthUser(struct Utilisateur* tete, int n) {
    if (n <= 0) {
        return NULL;
    }
    struct Utilisateur* current = tete;
    int count = 1;
    while (current != NULL && count < n) {
        current = current->suiv;
        count++;
    }
    return current;
}

int Voter(struct Utilisateur* user){
    Aff_condidats(condidat_tete);
    int n=Len(condidat_tete);
    int choix= Option_test(1,n);
    vote_tete=AjoutVote(vote_tete,user,GetNthUser(condidat_tete,choix));
}
int menu_Utilisateur(struct Utilisateur* user){
    printf("< Menu Utilisateur >\n\n");
    printf("(1) Participer au vote\n");
    printf("(2) Afficher les résultats de l'élection\n");
    printf("(0) Revenir au meun Principal\n");
    printf("\n");
    int login_input= Option_test(0,2);
    printf("*******\n");
    if(login_input==0) {
        Menu_principal();
    } else if(login_input==1) {
        Voter(user);
        menu_Utilisateur(user);
    } else if(login_input==2) {
        Show_res(vote_tete);
        menu_Utilisateur(user);
    }
    return 0;
}

int inscri_Admin(){
    struct Utilisateur admin;
    int confirmation_input=2;
    do{
    printf("< S'inscrire autant qu'un administrateur >\n\n");
    fgets(s, sizeof(s), stdin);
    printf("Saisir votre nom et prenom    : ");
    fgets(admin.nom_prenom, sizeof(admin.nom_prenom), stdin);
    admin.nom_prenom[strcspn(admin.nom_prenom, "\n")] = '\0';
    printf("Saisir le numero de votre CIN : ");
    fgets(admin.CIN, sizeof(admin.CIN), stdin);
    admin.CIN[strcspn(admin.CIN, "\n")] = '\0';
    printf("Choisir votre mot de passe    : ");
    fgets(admin.mot_de_passe, sizeof(admin.mot_de_passe), stdin);
    admin.mot_de_passe[strcspn(admin.mot_de_passe, "\n")] = '\0';
    printf("\n");
    printf("(1) Confirmer\n");
    printf("(2) Changer les données d'inscription\n\n");
    confirmation_input= Option_test(1,2);}
    while (confirmation_input != 1);
    admin.est_admin=1;
    users_tete=add_User(users_tete,&admin);
    printf("\nInsciption terminée aver succès\n\n");
    
    se_connecter();
    return 0;
}

int inscri_User(){
    struct Utilisateur user;
    int confirmation_input=2;
    do{
    printf("< S'inscrire autant qu'un utilisateur >\n\n");
    fgets(s, sizeof(s), stdin);
    printf("Saisir votre nom et prenom    : ");
    fgets(user.nom_prenom, sizeof(user.nom_prenom), stdin);
    user.nom_prenom[strcspn(user.nom_prenom, "\n")] = '\0';
    printf("Saisir le numero de votre CIN : ");
    fgets(user.CIN, sizeof(user.CIN), stdin);
    user.CIN[strcspn(user.CIN, "\n")] = '\0';
    printf("Choisir votre mot de passe    : ");
    fgets(user.mot_de_passe, sizeof(user.mot_de_passe), stdin);
    user.mot_de_passe[strcspn(user.mot_de_passe, "\n")] = '\0';
    printf("\n");
    printf("(1) Confirmer\n");
    printf("(2) Changer les données d'inscription\n\n");
    confirmation_input= Option_test(1,2);}
    while (confirmation_input != 1);
    user.est_admin=0;
    users_tete=add_User(users_tete,&user);
    printf("\nInsciption terminée aver succès\n\n");
    se_connecter();
    return 0;
}


int s_inscrire(){
    printf("< S'inscrire >\n\n");
    printf("(1) Administrateur\n");
    printf("(2) Utilisateur\n");
    printf("(0) Revenir au meun Principal\n");
    printf("\n");
    int signup_input= Option_test(0,2);
    printf("*******\n");
    if(signup_input==0) {
        Menu_principal();
    } else if(signup_input==1) {
        inscri_Admin();
    } else if(signup_input==2) {
        inscri_User();
    }
    return 0;
}
/*
int Verif_connection( struct Utilisateur* tete, struct Utilisateur* utilisateur) {
    const struct Utilisateur* courant = tete;

    while (courant != NULL) {
        int match = 1;
        for (int i = 0; i < 50; ++i) {
            if (courant->nom_prenom[i] != utilisateur->nom_prenom[i]) {
                match = 0;
                break;
            }
        }
        if (match) {
            for (int i = 0; i < 50; ++i) {
                if (courant->mot_de_passe[i] != utilisateur->mot_de_passe[i]) {
                    match = 0;
                    break;
                }
            }
        }
        if (match) {
            for (int i = 0; i < 10; ++i) {
                if (courant->CIN[i] != utilisateur->CIN[i]) {
                    match = 0;
                    break;
                }
            }
        }
        if (match) {
            return 1;
        }
        courant = courant->suiv;
    }
    return 0;
}*/
int Verif_connection(const struct Utilisateur* tete, const struct Utilisateur* utilisateur) {
    const struct Utilisateur* courant = tete;

    while (courant != NULL) {
        if (strcmp(courant->nom_prenom, utilisateur->nom_prenom) == 0 &&
            strcmp(courant->mot_de_passe, utilisateur->mot_de_passe) == 0 &&
            strcmp(courant->CIN, utilisateur->CIN) == 0 &&
            courant->est_admin == utilisateur->est_admin) {
            return 1;  
        }
        courant = courant->suiv;
    }
    return 0;  
}


int Connect_Admin(){
    struct Utilisateur admin;
    do{
    printf("< Se Connecter autant qu'un administrateur >\n\n");
    fgets(s, sizeof(s), stdin);
    printf("Saisir votre nom et prenom    : ");
    fgets(admin.nom_prenom, sizeof(admin.nom_prenom), stdin);
    admin.nom_prenom[strcspn(admin.nom_prenom, "\n")] = '\0';
    printf("Saisir le numero de votre CIN : ");
    fgets(admin.CIN, sizeof(admin.CIN), stdin);
    admin.CIN[strcspn(admin.CIN, "\n")] = '\0';
    printf("Choisir votre mot de passe    : ");
    fgets(admin.mot_de_passe, sizeof(admin.mot_de_passe), stdin);
    admin.mot_de_passe[strcspn(admin.mot_de_passe, "\n")] = '\0';
    admin.est_admin=1;
    printf("\n");
    if (Verif_connection (users_tete,&admin)!= 1) {
        printf("Verifier votre données de connection \n\n");
    }}
    while (Verif_connection (users_tete,&admin)!= 1);
    add_User(users_tete,&admin);
    printf("\nConnection avec succès\n\n");
    menu_admin();
    return 0;
}



int Connect_User(){
    struct Utilisateur user;
    do{
    printf("< Se Connecter autant qu'un utilisateur >\n\n");
    fgets(s, sizeof(s), stdin);
    printf("Saisir votre nom et prenom    : ");
    fgets(user.nom_prenom, sizeof(user.nom_prenom), stdin);
    user.nom_prenom[strcspn(user.nom_prenom, "\n")] = '\0';
    printf("Saisir le numero de votre CIN : ");
    fgets(user.CIN, sizeof(user.CIN), stdin);
    user.CIN[strcspn(user.CIN, "\n")] = '\0';
    printf("Choisir votre mot de passe    : ");
    fgets(user.mot_de_passe, sizeof(user.mot_de_passe), stdin);
    user.mot_de_passe[strcspn(user.mot_de_passe, "\n")] = '\0';
    user.est_admin=0;
    printf("\n");
    if (Verif_connection (users_tete,&user)!= 1) {
        printf("Verifier votre données de connection \n\n");
    }}
    while (Verif_connection (users_tete,&user)!= 1);
    
    add_User(users_tete,&user);
    printf("\nConnection avec succès\n\n");
    menu_Utilisateur(&user);
    return 0;
}


int se_connecter(){
    printf("< Se connecter >\n\n");
    printf("(1) Administrateur\n");
    printf("(2) Utilisateur\n");
    printf("(0) Revenir au meun Principal\n");
    printf("\n");
    int login_input= Option_test(0,2);
    printf("*******\n");
    if(login_input==0) {
        
        Menu_principal();
    } else if(login_input==1) {
        Connect_Admin();
    } else if(login_input==2) {
        Connect_User();
    }
    return 0;
}




void libererListeUtilisateurs(struct Utilisateur* tete) {
    struct Utilisateur* courant = tete;
    struct Utilisateur* suivant;

    while (courant != NULL) {
        suivant = courant->suiv;
        free(courant);
        courant = suivant;
    }
}

void libererListeVote(struct Vote* tete) {
    struct Utilisateur* courant = tete;
    struct Utilisateur* suivant;

    while (courant != NULL) {
        suivant = courant->suiv;
        free(courant);
        courant = suivant;
    }
}

void ReadWriteUsers(struct Utilisateur* tete) {
    FILE* file = fopen("users.bin", "wb");

    struct Utilisateur* current = tete;

    while (current != NULL) {
        fwrite(current, sizeof(struct Utilisateur), 1, file);
        current = current->suiv;
    }

    fclose(file);
}


void ReadWriteCondidat(struct Utilisateur* tete) {
    FILE* file = fopen("condidats.bin", "wb");

    struct Utilisateur* current = tete;

    while (current != NULL) {
        fwrite(current, sizeof(struct Utilisateur), 1, file);
        current = current->suiv;
    }

    fclose(file);
}


void ReadWriteVotes(struct Vote* tete) {
    FILE* file = fopen("votes.bin", "wb");

    struct Vote* current = tete;

    while (current != NULL) {
        fwrite(current, sizeof(struct Vote), 1, file);
        current = current->suiv;
    }

    fclose(file);
}

int Menu_principal(){
    printf("Bienvenu dans le meun Principal de E-Election\n\n(1) Se Connecter\n(2) S'inscrire\n(0) Quitter \n");
    int menu_input = Option_test(0,2);
    printf("*******\n");
    if(menu_input==1) {
        se_connecter();
    }else if(menu_input==2){
        s_inscrire();
    }else if(menu_input==0){
        printf("Vérification des données ");
        usleep(500000);
        printf("...\n");
        usleep(500000);
        printf("Changements sauvegardés ");
        usleep(700000);
        printf("... \n");
        usleep(300000);
        printf("-- 0% \n");
        usleep(600000);
        printf("-- 13% \n");
        usleep(100000);
        printf("-- 26% \n");
        usleep(500000);
        printf("-- 41% \n");
        usleep(200000);
        printf("-- 60% \n");
        usleep(600000);
        printf("-- 73% \n");
        usleep(100000);
        printf("-- 86% \n");
        usleep(500000);
        printf("-- 91% \n");
        usleep(200000);
        printf("-- 100% \n");
        usleep(300000);
        printf("Application fermée\n");
    }
    return 0;
}

int main() {
    //creer et ouvrir le fichiers
    users_tete= CreateOpenUsers();
    condidat_tete= CreateOpenCondidats();
    vote_tete= CreateOpenVotes();
    Menu_principal();
    //sauvegarder les liste dans les fichiers
    ReadWriteUsers(users_tete);
    ReadWriteCondidat(condidat_tete);
    ReadWriteVotes(vote_tete);
    //liberer la memoire
    libererListeUtilisateurs(users_tete);
    libererListeUtilisateurs(condidat_tete);
    libererListeVote(vote_tete);
    return 0;
}