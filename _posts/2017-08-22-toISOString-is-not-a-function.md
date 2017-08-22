

Most likely you're passing a React Synthetic Event, which cannot be serialized. See the similar issue and the troubleshooting. Note that the solution in that issue will not be available in React 16, since we're relying on React internals. So, I recommend to choose one of the solutions from the troubleshooting.

However, it would be great if we could make jsan not to throw in this case, but at least to skip those data. If you could add a pr with a test case there, it would be much appreciated. Seems like the issue with toISOString is only for specific events as I wasn't able to reproduce.


### see also:
* [toISOString is not a function](https://github.com/zalmoxisus/redux-devtools-extension/issues/315){:target="_blank"}
* [It fails to serialize data when passing synthetic events or calls an action directly with reduc-actins](https://github.com/zalmoxisus/redux-devtools-extension/blob/master/docs/Troubleshooting.md#it-fails-to-serialize-data-when-passing-synthetic-events-or-calling-an-action-directly-with-redux-actions){:target="_blank"}
