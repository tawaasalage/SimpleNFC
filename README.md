# SimpleNFC

SimpleNFC is a project which include insanely easy way of reading NFC data

### Tech

SimpleNFC uses a number of open source lib to work properly:

* [RXJava] - Reactive Extensions for the JVM
* [RXAndroid] - Reactive Extensions for Android

### Integrate for custom projects

Implement NFCInterface

```sh
public class MainActivity extends Activity implements NFCInterface{
....
```

Create Subscription local variable

```sh
    private Subscription nfcSubscription;
```

Fill the implemented methods

```sh
 @Override
    protected void onResume() {
        super.onResume();
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, new Intent(this,getClass()).addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP), 0);
        IntentFilter filter = new IntentFilter();
        filter.addAction(NfcAdapter.ACTION_TAG_DISCOVERED);
        filter.addAction(NfcAdapter.ACTION_NDEF_DISCOVERED);
        filter.addAction(NfcAdapter.ACTION_TECH_DISCOVERED);
        NfcAdapter nfcAdapter = NfcAdapter.getDefaultAdapter(this);
        nfcAdapter.enableForegroundDispatch(this, pendingIntent, new IntentFilter[]{filter}, this.techList);
    }

    @Override
    protected void onPause() {
        super.onPause();
        // disabling foreground dispatch:
        NfcAdapter nfcAdapter = NfcAdapter.getDefaultAdapter(this);
        nfcAdapter.disableForegroundDispatch(this);

    }

    @Override
    protected void onNewIntent(Intent intent) {
        if (intent.getAction().equals(NfcAdapter.ACTION_TAG_DISCOVERED)) {

            String type = intent.getType();
            Tag tag = intent.getParcelableExtra(NfcAdapter.EXTRA_TAG);
            nfcReader(tag);
        }
    }

    @Override
    public String nfcRead(Tag t) {
        try {
            Tag tag = t;
            Ndef ndef = Ndef.get(tag);
            if (ndef == null) {
                return null;
            }
            NdefMessage ndefMessage = ndef.getCachedNdefMessage();
            NdefRecord[] records = ndefMessage.getRecords();
            for (NdefRecord ndefRecord : records)
            {
                if (ndefRecord.getTnf() == NdefRecord.TNF_WELL_KNOWN && Arrays.equals(ndefRecord.getType(), NdefRecord.RTD_TEXT))
                {
                    try {return readText(ndefRecord);} catch (UnsupportedEncodingException e) {}
                }
            }
        }
        catch (Exception e)
        {
            return null;
        }
        return null;
    }

    @Override
    public String readText(NdefRecord record) throws UnsupportedEncodingException {
        byte[] payload = record.getPayload();
        String textEncoding = ((payload[0] & 128) == 0) ? "UTF-8" : "UTF-16";
        int languageCodeLength = payload[0] & 0063;
        return new String(payload, languageCodeLength + 1, payload.length - languageCodeLength - 1, textEncoding);
    }

    @Override
    public void nfcReader(Tag tag) {

        nfcSubscription= Observable.just(nfcRead(tag))
                .subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<String>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(String s) {
                        if (s != null) {

                            txtNFCID.setText(s);

                        }
                    }
                });
    }
```

Define onDestroy and unsubscribe on that

```sh
    @Override
    protected void onDestroy() {
        super.onDestroy();

        unsubscribe(nfcSubscription);

    }


    private static void unsubscribe(Subscription subscription) {
        if (subscription != null && !subscription.isUnsubscribed()) {
            subscription.unsubscribe();
            subscription = null;
        }
    }
```

