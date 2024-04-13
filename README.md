# Projet-c-

Création de la structure Pixel qui donne la teinte en rouge vert et bleu du pixel    
    
    using namespace std;

    struct Pixel {
        int r, g, b;                          

Création d'une classe coordonnées polaires définie par un rho et un theta
    
    class CoordPolaires {
    public:
        double theta;                         

        CoordPolaires(double t, double r) : theta(t), p(r) {}
    };

fonction qui crée une matrice nulle de taille choisie

    vector<vector<double>> matrice(int a,int b) {
        const int lignes = a;                                                               
        const int colonnes = b;

        vector<vector<double>> matrice(lignes, vector<double>(colonnes));                 
        for (int i=0;i<lignes;++i){
          for (int j=0;j<colonnes;++j){
            matrice[i][j] = 0;
          }
        }

        return matrice;
    }

Fonction qui lit un fichier .ppm

    vector<vector<Pixel>> lirePPM(string chemin) {                       
        ifstream fichier(chemin);                                       //ouverture du fichier choisi avec ifstream
        vector<vector<Pixel>> matrice;                                 //création d'une matrice vide pour la remplir des codes des pixels                                                     

        if (!fichier.is_open()) {
            cerr << "Impossible d'ouvrir le fichier : " << chemin << endl;          //message d'erreur si le fichier ne s'ouvre pas
            return matrice;
        }

        string format;                                                //création des variables qui définissent l'image comme le format (ici P3) sa taille et la valeur max des couleurs des pixels
        int largeur, hauteur, maxVal;
        fichier >> format >> largeur >> hauteur >> maxVal;

        if (format != "P3") {
            cerr << "Mauvais format" << endl;
            return matrice;                                            //message d'erreur qui dit que si le format n'est pas P3 on n'a pas le bon format
        }

        matrice.resize(hauteur, vector<Pixel>(largeur));              //redimensionner la matrice à la même taille que l'image

        for (int i = 0; i < hauteur; ++i) {                               
            for (int j = 0; j < largeur; ++j) {
                Pixel pixel;
                fichier >> pixel.r >> pixel.g >> pixel.b;                 //remplissage de la matrice des triplets étant les pixels
                matrice[i][j] = pixel;
            }
        }

        fichier.close();                                   //fermeture du fichier
        return matrice;
    }

Création de la classe transformée de Hough

    class transformHough {                                       //création de la classe transformée de Hough
    private:
      map<pair<int,int>,int> compteur;                          //map contenant les paires de rho theta et leurs itérations
    public:
      transformHough() {}
  
      void ajoutePoint(int x, int y) {
        for (int theta =0;theta<180;theta++) {
          float thetaRad = M_PI/180*theta;                                 //fonction ajoutant les doublets rho theta à la map et les compte 
          double p = x*cos(thetaRad) + y*sin(thetaRad);
          pair<int, int> pt = make_pair(theta, (int)p);
          if (compteur.find(pt) == compteur.end()) {
             compteur[pt] = 1;
          } else {
             compteur[pt]++;
            }
        }
      }

      void afficheCompteur() {
        for (const auto &pair : compteur) {
          if (pair.second>=250){
            cout << "Theta: " << pair.first.first << ", p: " << pair.first.second             //fonction affichant chaque paire de rho theta et leur nombre d'itérations
            << " -> Compte: " << pair.second << endl;                                    
          }
        }
      }
  
      vector<pair<int,int>> stock(int seuil){
        vector<pair<int,int>> vecteur;
        for (const auto &pair : compteur) {
          if (pair.second>=seuil){                                       //donne les rho theta ayant un compte supérieur à un seuil défini. On le choisira selon les valeurs qu'on obtient. Ici 
            vecteur.push_back(pair.first);                               // on prend 250 car 254 est le compteur le plus élevé et le deuxième est seulement 67
          }
        }
        return vecteur;
      }
  
    };

Fonction main
    
    int main() {
      transformHough TH;
  
      string chemin = "m1projetcpp2.ppm";
      vector<vector<Pixel>> matriceImage = lirePPM(chemin);                                //lecture du fichier proposé

      int rows = matriceImage.size();
      int cols = matriceImage[0].size();
  
      for (int i=0;i<rows;++i){
        for (int j=0;j<cols;++j){
          if (matriceImage[i][j].r > 0 || matriceImage[i][j].g > 0 || matriceImage[i][j].b > 0){            //pour chaque pixel de l'image , s'il est coloré on le rajoute au contour et on calcule son rho theta
            TH.ajoutePoint(i,j);
          }
        }
      }
      TH.afficheCompteur();
      vector<pair<int,int>> resultat = TH.stock(250);
      vector<vector<double>> MatResult = matrice(rows,cols);
  
      /*for (int i=0;i<resultat;i++){
        if (resultat[i][0] != 90){
          int theta = M_PI/180*resultat[i][0];
          int p = resultat[i][1];
      
        }*/

      return 0;
    }


fonctions supplémentaires implémentées mais non utiles au code ci-dessus



    CoordPolaires convertirEnCP(int a,int b){
      CoordPolaires polaires;
  
      polaires.p = sqrt(a*a+b*b);
      polaires.theta = atan2(b,a);
  
      return polaires;
    }

    void detectionContours(const vector<vector<double>>& Mat) {
      vector<CoordPolaires> points;

      for (int i = 0; i < Mat.size(); ++i) {
        for (int j = 0; j < Mat[i].size(); ++j) {
          if (Mat[i][j]==1){
            points.push_back(convertirEnCP(i,j));
          }
        }
      }
      cout << std::setw(10) << "Rayon" << std::setw(10) << "Theta" << endl;
        for (int i = 0; i < points.size(); ++i) {
          cout << std::setw(10) << points[i].p << std::setw(10) << points[i].theta << endl;
        }
      cout << endl;
    }
