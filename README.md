# Expo Cheat Sheet

## Quick Reference

Expo is a platform for building React Native applications with a managed workflow, providing tools, libraries, and cloud services.

### Setup and Installation

```bash
# Install Expo CLI
npm install -g @expo/cli

# Create new project
npx create-expo-app MyApp

# Start development server
cd MyApp && npx expo start

# Install dependencies
npx expo install package-name

# Build app
npx expo build:android
npx expo build:ios

# Publish update
npx expo publish
```

## Core Concepts

### App Configuration

```json
// app.json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "updates": {
      "fallbackToCacheTimeout": 0
    },
    "assetBundlePatterns": [
      "**/*"
    ],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.myapp"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "package": "com.yourcompany.myapp"
    },
    "web": {
      "favicon": "./assets/favicon.png"
    }
  }
}
```

### Basic App Structure

```javascript
// App.js
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, Text, View } from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
      <StatusBar style="auto" />
    </NavigationContainer>
  );
}

const HomeScreen = ({ navigation }) => (
  <View style={styles.container}>
    <Text>Welcome to Expo!</Text>
    <Button 
      title="Go to Details" 
      onPress={() => navigation.navigate('Details')} 
    />
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```

## Expo SDK Components

### Camera

```javascript
import { Camera } from 'expo-camera';
import { useState, useRef } from 'react';

const CameraScreen = () => {
  const [hasPermission, setHasPermission] = useState(null);
  const cameraRef = useRef(null);

  useEffect(() => {
    (async () => {
      const { status } = await Camera.requestCameraPermissionsAsync();
      setHasPermission(status === 'granted');
    })();
  }, []);

  const takePicture = async () => {
    if (cameraRef.current) {
      const photo = await cameraRef.current.takePictureAsync();
      console.log('Photo taken:', photo.uri);
    }
  };

  if (hasPermission === null) {
    return <View />;
  }
  if (hasPermission === false) {
    return <Text>No access to camera</Text>;
  }

  return (
    <View style={styles.container}>
      <Camera 
        style={styles.camera} 
        type={Camera.Constants.Type.back}
        ref={cameraRef}
      >
        <View style={styles.buttonContainer}>
          <TouchableOpacity style={styles.button} onPress={takePicture}>
            <Text style={styles.text}>Take Picture</Text>
          </TouchableOpacity>
        </View>
      </Camera>
    </View>
  );
};
```

### Location

```javascript
import * as Location from 'expo-location';
import { useState, useEffect } from 'react';

const LocationScreen = () => {
  const [location, setLocation] = useState(null);
  const [errorMsg, setErrorMsg] = useState(null);

  useEffect(() => {
    (async () => {
      let { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        setErrorMsg('Permission to access location was denied');
        return;
      }

      let location = await Location.getCurrentPositionAsync({});
      setLocation(location);
    })();
  }, []);

  const watchLocation = () => {
    Location.watchPositionAsync(
      {
        accuracy: Location.Accuracy.High,
        timeInterval: 1000,
        distanceInterval: 1,
      },
      (location) => {
        setLocation(location);
      }
    );
  };

  let text = 'Waiting...';
  if (errorMsg) {
    text = errorMsg;
  } else if (location) {
    text = JSON.stringify(location);
  }

  return (
    <View style={styles.container}>
      <Text>{text}</Text>
      <Button title="Watch Location" onPress={watchLocation} />
    </View>
  );
};
```

### Notifications

```javascript
import * as Notifications from 'expo-notifications';
import { useState, useEffect, useRef } from 'react';

// Configure notifications
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: false,
    shouldSetBadge: false,
  }),
});

const NotificationScreen = () => {
  const [expoPushToken, setExpoPushToken] = useState('');
  const notificationListener = useRef();
  const responseListener = useRef();

  useEffect(() => {
    registerForPushNotificationsAsync().then(token => setExpoPushToken(token));

    // Listen for notifications
    notificationListener.current = Notifications.addNotificationReceivedListener(notification => {
      console.log('Notification received:', notification);
    });

    responseListener.current = Notifications.addNotificationResponseReceivedListener(response => {
      console.log('Notification response:', response);
    });

    return () => {
      Notifications.removeNotificationSubscription(notificationListener.current);
      Notifications.removeNotificationSubscription(responseListener.current);
    };
  }, []);

  const schedulePushNotification = async () => {
    await Notifications.scheduleNotificationAsync({
      content: {
        title: "You've got mail! 📬",
        body: 'Here is the notification body',
        data: { data: 'goes here' },
      },
      trigger: { seconds: 2 },
    });
  };

  return (
    <View style={styles.container}>
      <Text>Your expo push token: {expoPushToken}</Text>
      <Button
        title="Press to schedule a notification"
        onPress={schedulePushNotification}
      />
    </View>
  );
};

async function registerForPushNotificationsAsync() {
  let token;
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;
  
  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }
  
  if (finalStatus !== 'granted') {
    alert('Failed to get push token for push notification!');
    return;
  }
  
  token = (await Notifications.getExpoPushTokenAsync()).data;
  console.log(token);

  return token;
}
```

### File System

```javascript
import * as FileSystem from 'expo-file-system';

const FileSystemExample = () => {
  const writeFile = async () => {
    const fileUri = FileSystem.documentDirectory + 'test.txt';
    await FileSystem.writeAsStringAsync(fileUri, 'Hello World!');
    console.log('File written to:', fileUri);
  };

  const readFile = async () => {
    const fileUri = FileSystem.documentDirectory + 'test.txt';
    try {
      const content = await FileSystem.readAsStringAsync(fileUri);
      console.log('File content:', content);
    } catch (error) {
      console.log('File not found');
    }
  };

  const downloadFile = async () => {
    const downloadUri = 'https://example.com/file.pdf';
    const fileUri = FileSystem.documentDirectory + 'downloaded_file.pdf';
    
    const { uri } = await FileSystem.downloadAsync(downloadUri, fileUri);
    console.log('Downloaded to:', uri);
  };

  const getFileInfo = async () => {
    const fileUri = FileSystem.documentDirectory + 'test.txt';
    const fileInfo = await FileSystem.getInfoAsync(fileUri);
    console.log('File info:', fileInfo);
  };

  return (
    <View style={styles.container}>
      <Button title="Write File" onPress={writeFile} />
      <Button title="Read File" onPress={readFile} />
      <Button title="Download File" onPress={downloadFile} />
      <Button title="Get File Info" onPress={getFileInfo} />
    </View>
  );
};
```

### Media Library

```javascript
import * as MediaLibrary from 'expo-media-library';
import { useState, useEffect } from 'react';

const MediaLibraryExample = () => {
  const [albums, setAlbums] = useState([]);
  const [photos, setPhotos] = useState([]);

  useEffect(() => {
    (async () => {
      const { status } = await MediaLibrary.requestPermissionsAsync();
      if (status === 'granted') {
        loadPhotos();
        loadAlbums();
      }
    })();
  }, []);

  const loadPhotos = async () => {
    const { assets } = await MediaLibrary.getAssetsAsync({
      first: 20,
      mediaType: 'photo',
    });
    setPhotos(assets);
  };

  const loadAlbums = async () => {
    const albumsData = await MediaLibrary.getAlbumsAsync();
    setAlbums(albumsData);
  };

  const savePhoto = async (uri) => {
    const asset = await MediaLibrary.createAssetAsync(uri);
    const album = await MediaLibrary.getAlbumAsync('MyApp');
    
    if (album == null) {
      await MediaLibrary.createAlbumAsync('MyApp', asset, false);
    } else {
      await MediaLibrary.addAssetsToAlbumAsync([asset], album, false);
    }
  };

  return (
    <View style={styles.container}>
      <Text>Albums: {albums.length}</Text>
      <Text>Photos: {photos.length}</Text>
      <FlatList
        data={photos}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <Image source={{ uri: item.uri }} style={{ width: 100, height: 100 }} />
        )}
        numColumns={3}
      />
    </View>
  );
};
```

## Over-the-Air Updates

### Publishing Updates

```bash
# Publish to default release channel
npx expo publish

# Publish to specific release channel
npx expo publish --release-channel staging

# Publish with specific message
npx expo publish --message "Bug fixes and improvements"
```

### Update Configuration

```javascript
// App.js - Check for updates
import * as Updates from 'expo-updates';

const App = () => {
  useEffect(() => {
    checkForUpdates();
  }, []);

  const checkForUpdates = async () => {
    try {
      const update = await Updates.checkForUpdateAsync();
      
      if (update.isAvailable) {
        await Updates.fetchUpdateAsync();
        // Optionally show alert to user
        Alert.alert(
          'Update Available',
          'A new update is available. Restart the app to apply it.',
          [
            { text: 'Later', style: 'cancel' },
            { text: 'Restart', onPress: () => Updates.reloadAsync() },
          ]
        );
      }
    } catch (error) {
      console.log('Error checking for updates:', error);
    }
  };

  // Rest of app
};
```

## Development Tools

### Expo Dev Tools

```bash
# Start with specific options
npx expo start --clear  # Clear cache
npx expo start --offline  # Work offline
npx expo start --web  # Start web version
npx expo start --android  # Start on Android
npx expo start --ios  # Start on iOS

# Tunnel for external access
npx expo start --tunnel

# Production mode
npx expo start --no-dev --minify
```

### Environment Variables

```javascript
// .env
API_URL=https://api.example.com
API_KEY=your-api-key

// Using environment variables
import Constants from 'expo-constants';

const apiUrl = Constants.manifest.extra?.apiUrl || 'https://api.example.com';

// Or with expo-constants
import { API_URL } from '@env';
```

## Building and Deployment

### EAS Build

```bash
# Install EAS CLI
npm install -g @expo/eas-cli

# Configure EAS
eas build:configure

# Build for Android
eas build --platform android

# Build for iOS
eas build --platform ios

# Build for both platforms
eas build --platform all

# Submit to app stores
eas submit --platform android
eas submit --platform ios
```

### EAS Build Configuration

```json
// eas.json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "android": {
        "buildType": "aab"
      }
    }
  },
  "submit": {
    "production": {
      "android": {
        "serviceAccountKeyPath": "../path/to/api-key.json",
        "track": "internal"
      },
      "ios": {
        "appleId": "your-apple-id@example.com",
        "ascAppId": "1234567890",
        "appleTeamId": "AB12CD34EF"
      }
    }
  }
}
```

## Ejecting and ExpoKit

### Ejecting to Bare Workflow

```bash
# Eject (now called expo eject)
npx expo eject

# This creates:
# - android/ folder with native Android code
# - ios/ folder with native iOS code
# - Allows custom native code modifications
```

### Custom Native Modules

```javascript
// After ejecting, you can add custom native modules
// Example: Custom module for Android

// android/app/src/main/java/.../CustomModule.java
public class CustomModule extends ReactContextBaseJavaModule {
  public CustomModule(ReactApplicationContext reactContext) {
    super(reactContext);
  }

  @Override
  public String getName() {
    return "CustomModule";
  }

  @ReactMethod
  public void showToast(String message) {
    Toast.makeText(getReactApplicationContext(), message, Toast.LENGTH_LONG).show();
  }
}

// Register in MainApplication.java
protected List<ReactPackage> getPackages() {
  return Arrays.<ReactPackage>asList(
    new MainReactPackage(),
    new CustomModulePackage()
  );
}

// Use in JavaScript
import { NativeModules } from 'react-native';
const { CustomModule } = NativeModules;

CustomModule.showToast('Hello from native module!');
```

## Testing

### Jest Testing

```javascript
// __tests__/App.test.js
import React from 'react';
import renderer from 'react-test-renderer';
import App from '../App';

describe('<App />', () => {
  it('has 1 child', () => {
    const tree = renderer.create(<App />).toJSON();
    expect(tree.children.length).toBe(1);
  });

  it('renders correctly', () => {
    const tree = renderer.create(<App />).toJSON();
    expect(tree).toMatchSnapshot();
  });
});
```

### Detox E2E Testing

```bash
# Install Detox
npm install -g detox-cli
npm install detox --save-dev

# Initialize Detox
detox init
```

```javascript
// e2e/firstTest.e2e.js
describe('Example', () => {
  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should have welcome screen', async () => {
    await expect(element(by.text('Welcome to Expo!'))).toBeVisible();
  });

  it('should show details after tap', async () => {
    await element(by.text('Go to Details')).tap();
    await expect(element(by.text('Details Screen'))).toBeVisible();
  });
});
```

## Performance Optimization

### Bundle Analysis

```bash
# Analyze bundle
npx expo export --dump-sourcemaps
npx react-native-bundle-visualizer

# Optimize assets
npx expo optimize
```

### Code Splitting

```javascript
// Lazy loading screens
import { lazy, Suspense } from 'react';
import { Text } from 'react-native';

const LazyScreen = lazy(() => import('./LazyScreen'));

const App = () => (
  <NavigationContainer>
    <Stack.Navigator>
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen 
        name="Lazy" 
        component={() => (
          <Suspense fallback={<Text>Loading...</Text>}>
            <LazyScreen />
          </Suspense>
        )} 
      />
    </Stack.Navigator>
  </NavigationContainer>
);
```

### Image Optimization

```javascript
// Using expo-image for better performance
import { Image } from 'expo-image';

const OptimizedImage = () => (
  <Image
    source={{ uri: 'https://example.com/image.jpg' }}
    style={{ width: 200, height: 200 }}
    contentFit="cover"
    cachePolicy="memory-disk"
    placeholder={require('./placeholder.png')}
  />
);
```

## Official Resources

- [Expo Documentation](https://docs.expo.dev/)
- [Expo SDK Reference](https://docs.expo.dev/versions/latest/)
- [Expo Snacks](https://snack.expo.dev/)
- [Expo GitHub](https://github.com/expo/expo)
- [EAS Documentation](https://docs.expo.dev/eas/)
- [Expo Forums](https://forums.expo.dev/)

## Community and Tools

- [Awesome Expo](https://github.com/expo/awesome-expo)
- [Expo Discord](https://discord.gg/4gtbPAdpaE)
- [Expo Reddit](https://www.reddit.com/r/expo/)
- [Expo Templates](https://github.com/expo/examples)
- [React Native Directory](https://reactnative.directory/)
- [Expo Blog](https://blog.expo.dev/)