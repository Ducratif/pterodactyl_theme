# pterodactyl_theme

VPS HOME
/var/www/pterodactyl/resources/scripts/assets/custom.css


import:
/var/www/pterodactyl/resources/scripts/index.tsx



Voici un **`index.tsx` complet** avec tout ce qu’il faut pour activer :

* la **couche de particules** (`.du-particles`)
* l’**effet spotlight** qui suit la souris sur les cartes serveurs (`.server-card`)
* l’**effet ripple** sur tous les boutons/cliquables (coordonnées du clic → variables CSS `--x / --y`)

Colle **exactement** ceci dans `/var/www/pterodactyl/resources/scripts/index.tsx`, à la place de ton fichier actuel.

```tsx
// Ducratif — theme override (CSS packagé)
import './assets/custom.css';

import React, { Fragment, useEffect } from 'react';
import ReactDOM from 'react-dom';
import App from '@/components/App';
import { setConfig } from 'react-hot-loader';

// Enable language support.
import './i18n';

// Empêche les reloads React Hot Loader sur les hooks
setConfig({ reloadHooks: false });

/**
 * Hook global UI: active les particules, le spotlight sur .server-card
 * et le ripple sur les boutons/cliquables.
 */
const useGlobalUIEffects = () => {
  useEffect(() => {
    // 1) Couche de particules (si pas déjà présente)
    if (!document.querySelector('.du-particles')) {
      const particles = document.createElement('div');
      particles.className = 'du-particles';
      document.body.appendChild(particles);
    }

    // 2) Spotlight sur cartes serveurs
    const onSpotlightMove = (e: MouseEvent) => {
      // On ne calcule que si la cible est dans une .server-card (ou la card elle-même)
      const target = (e.target as HTMLElement) || null;
      const card = target?.closest?.('.server-card') as HTMLElement | null;
      if (!card) return;

      const rect = card.getBoundingClientRect();
      const mx = ((e.clientX - rect.left) / rect.width) * 100;
      const my = ((e.clientY - rect.top) / rect.height) * 100;
      // Variables CSS consommées par custom.css (server-card::before)
      card.style.setProperty('--mx', `${mx}%`);
      card.style.setProperty('--my', `${my}%`);
    };

    // 3) Ripple coordonnées sur clic (pour tous les boutons/cliquables)
    const onRipple = (e: MouseEvent) => {
      const el = (e.target as HTMLElement) || null;
      if (!el) return;

      // On remonte si l’élément cliqué n'est pas un bouton direct
      const clickable = el.closest('button, .btn, [role="button"]') as HTMLElement | null;
      if (!clickable) return;

      const rect = clickable.getBoundingClientRect();
      const x = ((e.clientX - rect.left) / rect.width) * 100;
      const y = ((e.clientY - rect.top) / rect.height) * 100;

      // --x / --y exploitées par custom.css (button::after gradient radial)
      clickable.style.setProperty('--x', `${x}%`);
      clickable.style.setProperty('--y', `${y}%`);
    };

    // 4) Optionnel: améliore les focus visibles au clavier
    const onKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Tab') {
        document.documentElement.classList.add('du-keyboard-nav');
      }
    };

    document.addEventListener('mousemove', onSpotlightMove, { passive: true });
    document.addEventListener('mousedown', onRipple, { passive: true });
    document.addEventListener('keydown', onKeyDown, { passive: true });

    return () => {
      document.removeEventListener('mousemove', onSpotlightMove);
      document.removeEventListener('mousedown', onRipple);
      document.removeEventListener('keydown', onKeyDown);
    };
  }, []);
};

// Petit wrapper pour brancher les effets globaux proprement
const RootWithEffects: React.FC = () => {
  useGlobalUIEffects();
  return (
    <Fragment>
      {/* La couche particules est injectée dans <body> via useEffect */}
      <App />
    </Fragment>
  );
};

ReactDOM.render(<RootWithEffects />, document.getElementById('app'));
```

### Ensuite

* Rebuild :

  * Avec Node 18 (flag OpenSSL) :

    ```bash
    cd /var/www/pterodactyl
    export NODE_OPTIONS=--openssl-legacy-provider
    yarn build:production
    php artisan view:clear
    ```
  * ou avec **nvm Node 16** (pas besoin du flag).

* Hard refresh sur le panel (**Ctrl+F5**).
