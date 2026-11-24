voici les tutos

<div id="ifixit-app" style="margin-top: 20px; font-family: inherit;">
  <!-- En-tête et partenaire -->
  <div style="display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 15px; margin-bottom: 15px;">
    <p style="margin: 0; color: #444; font-size: 0.95rem;">
      Recherchez parmi des milliers de tutoriels de réparation gratuits grâce à la base de données <strong>iFixit</strong> :
    </p>
    <span style="font-weight: 800; color: #0071ce; font-size: 1.1rem; letter-spacing: 0.5px;">
      ✕ IFIXIT
    </span>
  </div>

  <!-- Barre de recherche principale -->
  <form id="ifixit-form" style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <input type="text" id="ifixit-query" placeholder="Tapez le nom ou la marque du produit (ex: aspirateur, Senseo, Dyson...)" 
           value="aspirateur"
           style="flex: 1; min-width: 240px; padding: 10px 14px; border: 1px solid #ccc; border-radius: 4px; font-size: 0.95rem;">
    <button type="submit" 
            style="padding: 10px 24px; background-color: #ee6c4d; color: white; border: none; border-radius: 4px; font-weight: bold; cursor: pointer; font-size: 0.95rem;">
      Rechercher
    </button>
  </form>

  <!-- Options de tri et filtres rapides -->
  <div style="display: flex; flex-wrap: wrap; gap: 10px; margin-bottom: 20px;">
    <select id="filter-marque" style="flex: 1; min-width: 150px; padding: 8px; border: 1px solid #ddd; border-radius: 4px; background: #f9f9f9;">
      <option value="">Filtrer par marque...</option>
      <option value="Dyson">Dyson</option>
      <option value="Philips">Philips</option>
      <option value="Miele">Miele</option>
      <option value="Bosch">Bosch</option>
      <option value="Krups">Krups</option>
      <option value="Samsung">Samsung</option>
      <option value="Apple">Apple</option>
      <option value="Nintendo">Nintendo</option>
    </select>

    <select id="filter-panne" style="flex: 1; min-width: 150px; padding: 8px; border: 1px solid #ddd; border-radius: 4px; background: #f9f9f9;">
      <option value="">Type de panne / pièce...</option>
      <option value="batterie">Batterie / Alimentation</option>
      <option value="moteur">Moteur</option>
      <option value="écran">Écran / Vitre</option>
      <option value="câble">Câble / Cordon</option>
      <option value="roulement">Roulement / Roue</option>
      <option value="bouton">Bouton / Interrupteur</option>
    </select>

    <select id="sort-order" style="flex: 1; min-width: 150px; padding: 8px; border: 1px solid #ddd; border-radius: 4px; background: #f9f9f9;">
      <option value="relevance">Trier par : Pertinence</option>
      <option value="az">Trier par : Titre (A → Z)</option>
      <option value="za">Trier par : Titre (Z → A)</option>
    </select>
  </div>

  <!-- Compteur de résultats -->
  <div id="results-count" style="font-size: 0.85rem; color: #666; margin-bottom: 15px; font-weight: 500;">
    Chargement des tutoriels...
  </div>

  <!-- Grille de résultats (4 colonnes sur ordinateur, responsive sur mobile) -->
  <div id="ifixit-grid" style="display: grid; grid-template-columns: repeat(auto-fill, minmax(210px, 1fr)); gap: 18px;">
  </div>
</div>

<script>
document.addEventListener("DOMContentLoaded", () => {
  const form = document.getElementById("ifixit-form");
  const queryInput = document.getElementById("ifixit-query");
  const filterMarque = document.getElementById("filter-marque");
  const filterPanne = document.getElementById("filter-panne");
  const sortOrder = document.getElementById("sort-order");
  const countDisplay = document.getElementById("results-count");
  const grid = document.getElementById("ifixit-grid");

  let currentResults = [];

  // Fonction d'appel à l'API publique d'iFixit
  async function rechercherTutos() {
    const motCle = queryInput.value.trim();
    const marque = filterMarque.value;
    const panne = filterPanne.value;

    // Construction de la requête combinée
    const rechercheFinale = [motCle, marque, panne].filter(Boolean).join(" ");
    
    if (!rechercheFinale) {
      countDisplay.textContent = "Veuillez saisir un mot-clé ou choisir un filtre.";
      grid.innerHTML = "";
      return;
    }

    countDisplay.textContent = "Recherche en cours sur iFixit...";
    grid.innerHTML = "";

    try {
      const url = `https://fr.ifixit.com/api/2.0/search/${encodeURIComponent(rechercheFinale)}?filter=guide&limit=24`;
      const response = await fetch(url);
      const data = await response.json();

      currentResults = data.results || [];
      const total = data.totalResults || currentResults.length;

      if (currentResults.length === 0) {
        countDisplay.textContent = "0 résultat trouvé. Essayez un terme plus général (ex: cafetière, aspirateur).";
        return;
      }

      countDisplay.textContent = `1 - ${currentResults.length} résultats affichés sur ${total} disponibles`;
      afficherGrille();
    } catch (error) {
      console.error("Erreur API iFixit :", error);
      countDisplay.innerHTML = `<span style="color: #d9534f;">Impossible de contacter iFixit pour le moment.</span>`;
    }
  }

  // Tri et affichage des cartes en grille
  function afficherGrille() {
    let liste = [...currentResults];

    if (sortOrder.value === "az") {
      liste.sort((a, b) => a.title.localeCompare(b.title));
    } else if (sortOrder.value === "za") {
      liste.sort((a, b) => b.title.localeCompare(a.title));
    }

    grid.innerHTML = liste.map(guide => {
      // Récupération de la miniature haute qualité ou image par défaut
      const imageUrl = (guide.image && (guide.image.standard || guide.image.medium || guide.image.thumbnail)) 
                       || "https://via.placeholder.com/300x225?text=Tutoriel+iFixit";

      return `
        <a href="${guide.url}" target="_blank" rel="noopener noreferrer" 
           style="text-decoration: none; color: inherit; display: flex; flex-direction: column; transition: transform 0.15s ease;">
          <div style="width: 100%; aspect-ratio: 4 / 3; overflow: hidden; border-radius: 4px; background: #eee; border: 1px solid #e0e0e0;">
            <img src="${imageUrl}" alt="${guide.title}" loading="lazy"
                 style="width: 100%; height: 100%; object-fit: cover; display: block;">
          </div>
          <div style="margin-top: 8px; font-size: 0.88rem; font-weight: 600; line-height: 1.3; color: #143258;">
            ${guide.title}
          </div>
        </a>
      `;
    }).join("");
  }

  // Événements
  form.addEventListener("submit", (e) => {
    e.preventDefault();
    rechercherTutos();
  });

  filterMarque.addEventListener("change", rechercherTutos);
  filterPanne.addEventListener("change", rechercherTutos);
  sortOrder.addEventListener("change", afficherGrille);

  // Lancement automatique d'une recherche initiale au chargement de la page
  rechercherTutos();
});


_____________________________________________________________________________________________________________________________

<img width="235" height="51" alt="image" src="https://github.com/user-attachments/assets/d2c75174-9974-4527-b0ab-0d278e767982" />


  
Suivez nous sur Instagram, Facebook


  

[<img width="49" height="55" alt="image" src="https://github.com/user-attachments/assets/763ac71e-d0fd-4200-bccf-fa1940ea6ccd" />
](https://www.instagram.com/repair_cafe_cuincy/)
[<img width="49" height="53" alt="image" src="https://github.com/user-attachments/assets/d50cad48-2ea7-47f4-8526-77e2774c3a04" />
](https://www.facebook.com/groups/734024094957742/)

