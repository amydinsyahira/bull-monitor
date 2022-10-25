# @amydin/bull-monitor-express

[Express](https://github.com/expressjs/express) adapter for [bull-monitor](https://github.com/amydinsyahira/bull-monitor)

## Usage

```sh
npm i @amydin/bull-monitor-express
```

```typescript
import { BullMonitorExpress } from '@amydin/bull-monitor-express';
import { BullAdapter } from '@amydin/bull-monitor-root/dist/bull-adapter';
// for BullMQ users
// import { BullMQAdapter } from "@amydin/bull-monitor-root/dist/bullmq-adapter";
import Express from 'express';
import Queue from 'bull';
// for Redis adapter
import KeyvRedis from '@keyv/redis';
// import Redis from 'ioredis';
// for Memcache adapter
// import KeyvMemcache from '@keyv/memcache';

(async () => {
  const app = Express();
  // manually create a storage adapter instance
  const keyvRedis = new KeyvRedis('redis://user:pass@localhost:6379');
  // Or reuse a previous Redis instance
  // const redis = new Redis('redis://user:pass@localhost:6379');
  // const keyvRedis = new KeyvRedis(redis);
  // for Memcache
  // const memcache = new KeyvMemcache('user:pass@localhost:11211');
  const monitor = new BullMonitorExpress({
    queues: [
      new BullAdapter(new Queue('1', 'REDIS_URI')),
      // readonly queue
      new BullAdapter(new Queue('2', 'REDIS_URI'), { readonly: true }),
    ],
    // enables graphql introspection query. false by default if NODE_ENV == production, true otherwise
    gqlIntrospection: true,
    // configure the cache in a manner that isn't unbounded (which is current default behavior). 
    // this protects your server from attacks that exhaust available memory, causing a DOS (adapter: keyvRedis, memcache, etc [https://github.com/jaredwray/keyv#storage-adapters])
    gqlCache: { adapter: keyvRedis } || 'bounded' || false,
    // enable metrics collector. false by default
    // metrics are persisted into redis as a list
    // with keys in format "bull_monitor::metrics::{{queue}}"
    metrics: {
      // collect metrics every X
      // where X is any value supported by https://github.com/kibertoad/toad-scheduler
      collectInterval: { hours: 1 },
      maxMetrics: 100,
      // disable metrics for specific queues
      blacklist: ['1'],
    },
  });
  await monitor.init();
  app.use('/my/url', monitor.router);
  app.listen(3000);

  // replace queues
  monitor.setQueues([new BullAdapter(new Queue('3', 'REDIS_URI'))]);
})();
```
