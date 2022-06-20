GTM + SPA hook

```js
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

const useTracking = () => {
  const location = useLocation();

  useEffect(() => {
    const unlisten = () => {
      if (!window.dataLayer) return;
      window.dataLayer.push({
        event: 'pageview',
        page: {
          path: location.pathname,
        },
      });
    };
    return unlisten;
  }, [location]);
};

export default useTracking;
```