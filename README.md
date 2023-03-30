# Google_Map
How to use Google map in your React.js app



How to use Google map in your app

install `@react-google-maps/api for google map`

intsall `react-google-autocomplete/lib/usePlacesAutocompleteService`

create a component `Google map` 

```js
import { useLoadScript } from '@react-google-maps/api';
import Loading from '../Loading';
import Map from './Map';
import styles from './Map.module.scss';

interface Props {
  autocompleteSearchInput: any;
  userLatLng: any;
  autocompleteSearchInputValue: (values: any) => void;
  userCurrentLocation: (lat: number, lng: number) => void;
}

const GoogleMap: React.FC<Props> = ({ autocompleteSearchInput, userLatLng, autocompleteSearchInputValue, userCurrentLocation }) => {
  const { isLoaded } = useLoadScript({
    googleMapsApiKey: "Your_API_key",
    libraries: ['places'],
  });
  if (!isLoaded) return <Loading loading={true} />;
  return (
    <>
      {isLoaded && (
        <div className={styles.map}>
          <Map
            isLoaded={isLoaded}
            userCurrentLocation={userCurrentLocation}
            autocompleteSearchInputValue={autocompleteSearchInputValue}
            autocompleteSearchInput={autocompleteSearchInput}
            userLatLng={userLatLng}
          />
        </div>
      )}
    </>
  );
};

export default GoogleMap;

```


Rape `Map` component in `Googe Map` 


```js
import React, { useEffect, useRef, useState } from 'react';
import { GoogleMap, MarkerF, useGoogleMap, useJsApiLoader } from '@react-google-maps/api';
import styles from './Map.module.scss';
import { AimOutlined } from '@ant-design/icons';
import CurrentLocation from '../CurrentLocation';
import Loading from '../Loading';
import { LoadingOutlined } from '@ant-design/icons';
import { Spin } from 'antd';

interface Props {
  isLoaded: any;
  autocompleteSearchInput: any;
  userLatLng: any;
  autocompleteSearchInputValue: (values: any) => void;
  userCurrentLocation: (lat: number, lng: number) => void;
}
const Map: React.FC<Props> = ({ isLoaded, autocompleteSearchInput, userLatLng, autocompleteSearchInputValue, userCurrentLocation }) => {
  const [selected, setSelected] = useState(false);
  const [loading, setLoading] = useState(false);
  const geocoder = new google.maps.Geocoder();
  const refMap = useRef<any>(null);
  const [center, setCenter] = useState<any>({
    lat: "", lng: ""
  });
  const antIcon = <LoadingOutlined style={{ fontSize: 24 }} spin />;

  const handleBoundsChanged = () => {
    console.log("yes");
    const mapCenterLat = refMap.current.getCenter().lat();
    const mapCenterLng = refMap.current.getCenter().lng();
    userCurrentLocation(mapCenterLat, mapCenterLng)
  };

  useEffect(() => {
    geocodeLatLng(geocoder)
    setCenter((prev: any) => {
      return {
        ...prev,
        lat: userLatLng.lat,
        lng: userLatLng.lng
      }
    })
  }, [userLatLng]);

  function geocodeLatLng(geocoder: google.maps.Geocoder) {
    geocoder
      .geocode({ location: userLatLng })
      .then((response) => {
        if (response.results[0]) {
          autocompleteSearchInputValue(response.results[0].formatted_address);
          setSelected(false)
        } else {
          console.log('No results found');
        }
      })
      .catch((e) => console.log('Geocoder failed due to: ' + e));
  }

  return (
    <>
      {!isLoaded && <Loading loading={true} />}
      {isLoaded && (
        <GoogleMap
          ref={refMap}
          zoom={18}
          center={center}
          onLoad={(map) => {
            refMap.current = map;
          }}
          options={{ streetViewControl: false }}
          mapContainerClassName={styles.mapContainer}
          onDragEnd={handleBoundsChanged}
        >
          <div className={styles.marker}>
          </div>
          <div
            style={{
              position: 'absolute',
              zIndex: '1',
              bottom: '150px',
              right: '40px',
              cursor: 'pointer',
              width: '10px',
              height: '18px',
            }}
          >
            <button
              style={{
                border: 'none',
                background: '#fff',
                padding: '8px',
                textAlign: 'center',
              }}
              onClick={() => {
                setSelected(true);
                setLoading(true);
              }}
            >
              {!selected ?
                <AimOutlined className={styles.aimIcon} />
                : loading ? <Spin indicator={antIcon} className={styles.aimIcon} />
                  : ""
              }
            </button>
            {selected && <CurrentLocation userCurrentLocation={userCurrentLocation} />}
          </div>
        </GoogleMap>
      )}
    </>
  );
};

export default Map;

```


Search bar for Search Location and autocomplete Suggestions 

create a component `AutocompleteSearch`


```js
import { useEffect, useState } from 'react';
import { Button, Input } from 'antd';
import styles from './Search.module.scss';
import { SearchOutlined, CloseOutlined, AimOutlined } from '@ant-design/icons';
import useGoogle from 'react-google-autocomplete/lib/usePlacesAutocompleteService';
import CurrentLocation from '../../components/CurrentLocation';
import { LoadingOutlined, ArrowRightOutlined } from '@ant-design/icons';
import { Spin } from 'antd';
import { Link } from 'react-router-dom';

interface Props {
  userCurrentLocation: (lat: number, lng: number) => void;
  userLatLng: any;
  googleMapModel: (value: boolean) => void;
  closeIcon?: boolean;
  sideBorder?: boolean;
  searchIcon?: boolean;
  arrowButton?: boolean;
  aimIcon?: boolean;
  deliveryButton?: boolean;
  autocompleteSearchInputValue?: (values: any) => void;
  autocompleteSearchInput?: any;
}

export const AutocompleteSearch: React.FC<Props> = ({
  userCurrentLocation,
  userLatLng,
  googleMapModel,
  searchIcon,
  closeIcon,
  aimIcon,
  sideBorder,
  deliveryButton,
  arrowButton,
  autocompleteSearchInputValue,
  autocompleteSearchInput,
}) => {
  const { placePredictions, getPlacePredictions, isPlacePredictionsLoading } =
    useGoogle({
      apiKey: "YOUR_GOOGLE_MAP_API",
    });

  const [aim, setAim] = useState(false);
  const [isOpen, setIsOpen] = useState(false);
  const [loading, setLoading] = useState(false);
  const [isMapOpen, setIsMapOpen] = useState(false)
  const geocoder = new google.maps.Geocoder();
  const [address, setAddress] = useState<any>();
  const antIcon = <LoadingOutlined style={{ fontSize: 24 }} spin />;

  const handelClick = () => {
    setIsOpen(false);
    autocompleteSearchInputValue?.("");
  };

  useEffect(() => {
    geocodeLatLng(geocoder);
  }, [userLatLng]);

  function geocodeLatLng(geocoder: google.maps.Geocoder) {
    geocoder
      .geocode({ location: userLatLng })
      .then((response) => {
        if (response.results[0]) {
          autocompleteSearchInputValue?.(response.results[0].formatted_address);
          setLoading(false)
          if (isMapOpen) {
            googleMapModel(true);
            setIsMapOpen(false)
          }
          setAim(false)
        } else {
          console.log('No results found');
        }
      })
      .catch((e) => console.log('Geocoder failed due to: ' + e));
  }

  useEffect(() => {
    if (address) {
      userCurrentLocation(address.lat, address.lng);
    }
  }, [address])

  function geocode(request: google.maps.GeocoderRequest): void {
    geocoder
      .geocode(request)
      .then((response) => {
        if (response.results[0]) {
          setAddress(response.results[0].geometry.location.toJSON());
        }
      })
      .catch((e) => console.log('Geocoder failed due to: ' + e));
  }

  return (
    <div className={styles.autocomplete}>
      <div className={styles.searchInput}>
        <Input
          value={autocompleteSearchInput}
          className={styles.input}
          placeholder='Enter address'
          onChange={(evt) => {
            getPlacePredictions({ input: evt.target.value });
            autocompleteSearchInputValue?.(evt.target.value);
            if (autocompleteSearchInput) {
              setIsOpen(true);
            } else {
              setIsOpen(false);
            }
          }}
        />
        {searchIcon && <SearchOutlined className={styles.searchIcon} />}
        {
          isOpen && autocompleteSearchInput && closeIcon ?
            <CloseOutlined className={styles.clearIcon} onClick={handelClick} />
            : loading ?
              <Spin indicator={antIcon} className={styles.clearIcon} />
              : ""
        }
        {aimIcon && (
          <AimOutlined
            className={styles.locationIcon}
            onClick={() => {
              setAim(true);
              setIsOpen(false);
              setLoading(true);
            }}
          />
        )}
        {sideBorder &&
          <div className={styles.sideBorder}></div>
        }
        {deliveryButton && (
          <div style={{ margin: 'auto 32px' }}>
            <Button
              type="primary"
              onClick={() => {
                autocompleteSearchInput ?
                  geocode({
                    address: autocompleteSearchInput,
                  })
                  : setAim(true)
                setIsOpen(false);  
                setLoading(true);
                setIsMapOpen(true);
              }}
            >
              Delivery
            </Button>
          </div>
        )}
        {arrowButton && (
          <div style={{ margin: 'auto 32px' }}>
            <Link to="/search">
              <Button
                type="primary"
                onClick={() => {
                }}
              >
                <ArrowRightOutlined />
              </Button>
            </Link>
          </div>
        )
        }
        {aim && <CurrentLocation userCurrentLocation={userCurrentLocation} />}
      </div>

      {isOpen && (
        <div className={styles.suggestionsModel}>
          {placePredictions.map((places, i) => {
            return (
              <ul className={styles.list}>
                <li
                  key={i}
                  style={{ cursor: 'pointer', textAlign: 'start' }}
                  onClick={() => {
                    autocompleteSearchInputValue?.(places.description);
                    if (places.description) {
                      geocode({
                        address: autocompleteSearchInput,
                      });
                    }
                    setIsOpen(false);
                  }}
                >
                  {places.description}
                </li>
              </ul>
            );
          })}
        </div>
      )}
    </div>
  );
};

export default AutocompleteSearch;
```


create a `Current Location` component

```js
import React, { useEffect, useState } from 'react';

interface Props {
  userCurrentLocation: (lat: number, lng: number) => void;
}

const CurrentLocation: React.FC<Props> = ({ userCurrentLocation }) => {
  const successCallback = (position: any) => {
    userCurrentLocation(position.coords.latitude, position.coords.longitude);
  };

  const errorCallback = (error: any) => {
    console.log(error);
  };

  const userLocation = () => {
    navigator.geolocation.getCurrentPosition(successCallback, errorCallback);
  };

  useEffect(() => {
    userLocation();
  }, []);
  return null;
};

export default CurrentLocation;

```


styles of `Google Map and Map component`

```scss
.map {
  height: 80%;
  width: 100%;
  // border: yellow solid 2px;
  // overflow-y: scroll;

  @media screen and (min-width: 992px) {
    height: 75%;

  }
}

.mapContainer {
  height: 100%;
  width: 100%;
  position: relative;

  .aimIcon {
    font-size: 24px;
    cursor: pointer;

    svg {
      fill: #666666;
      transition: 0.2s linear;

      &:hover {
        fill: #000;
      }
    }
  }

  .marker{
    width: 22px;
    height: 40px;
    display: block;
    content: ' ';
    position: absolute;
    top: 50%; left: 50%;
    margin: -40px 0 0 -11px;
    background: url('https://maps.gstatic.com/mapfiles/api-3/images/spotlight-poi_hdpi.png');
    background-size: 22px 40px; /* Since I used the HiDPI marker version this compensates for the 2x size */
    pointer-events: none; /* This disables clicks on the marker. Not fully supported by all major browsers, though */
  }
}
```

styles of `AutocopleteSearch`

```scss
.autocomplete {
  background-color: #f0f0f6;
  width: 100%;
  border-radius: 10px;
  position: relative;

  @media screen and (min-width: 992px) {
    width: 70%;
    margin: 0 auto;
  }

  .searchInput {
    width: 100%;
    margin-top: 16px;
    display: flex;
    align-items: center;
    position: relative;
    // max-width: 800px;

    @media screen and (min-width: 992px) {
      width: 100%;
    }

    .input {
      width: 100%;
      padding: 20px 56px;
      border: none;
      border-radius: 10px;
      color: #444;
      background-color: #f0f0f6;
      font-size: 16px;
      font-weight: 300;

      &::-webkit-input-placeholder {
        /* Edge */
        color: #808080;
      }

      &:-ms-input-placeholder {
        /* Internet Explorer 10-11 */
        color: #808080;
      }

      &::placeholder {
        color: #808080;
      }
      &:focus{
        box-shadow: none;
      }
    }

    .locationIcon {
      right: 14px;
      top: auto;
      font-size: 20px;
      cursor: pointer;

      svg {
        fill: #808080;
        transition: 0.2s linear;

        &:hover {
          fill: #000;
        }
      }

    }

    .sideBorder{
      border-left: 2px solid #808080;
      height: 25px;
      margin-left: 30px;
    }

    .searchIcon,
    .clearIcon {
      padding: 8px;
      min-width: 64px;
      position: absolute;
      font-size: 20px;

      svg {
        fill: #808080;
      }
    }

    .searchIcon {
      left: 0px;
    }

    .clearIcon {
      right: 200px;
    }

  }

  .suggestionsModel {
    background: #f0f0f6;
    height: max-content;
    border-radius: 10px;
    overflow-y: auto;
    position: absolute;
    z-index: 1;
    width: 100%;
  }

  .list {
    list-style: none;
    padding: 10px 56px;
    font-size: 16px;
    color: #808080;
  }

}
```
