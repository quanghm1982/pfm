# pfm
This is testing
# Native
import * as React from 'react';
import { WebView } from 'react-native-webview';

export default class App extends React.Component {
  render() {
    return <WebView source={{ uri: 'https://expo.io' }} style={{ marginTop: 20 }} />;
  }
}
