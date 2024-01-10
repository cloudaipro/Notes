<code>
validation_size = 0.2
train_size = int(len(X) * (1-validation_size))
X_train, X_test = X[0:train_size], X[train_size:len(X)]
Y_train, Y_test = Y[0:train_size], Y[train_size:len(X)]
</code>

<b>LSTM</b> 
<code>
Y_train_LSTM, Y_test_LSTM = np.array(Y_train)[seq_len-1:], np.array(Y_test)
X_train_LSTM = np.zeros((X_train.shape[0]-seq_len+1, seq_len, X_train.shape[1]))
X_test_LSTM  = np.zeros((X_test.shape[0], seq_len, X.shape[1]))
for i in range(seq_len):
    X_train_LSTM[:,i,:] = np.array(X_train)[i:X_train.shape[0]-seq_len+1+i, :]
    X_test_LSTM[:,i,: ]= np.array(X)[X_train.shape[0]-seq_len+i+1 : X.shape[0]-seq_len+1+i, :]
</code>
