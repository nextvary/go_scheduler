# go_scheduler
go_scheduler

```go
scheduler.NewScheduler(5, 10, "money", nil).Start(yieldFunc, workFunc)


func yieldFunc(s *scheduler.Scheduler) {
	var lastId int64
	dealNums := 0
	limit := 10
	for {
		//查询表
		list, err := dao.GetUserMoneyList(lastId, limit)
		if err != nil {
			dlog.Error("获取用户元宝列表失败", err)
			return
		}
		for _, userMoney := range list {
			dealNums++
			if config.IsDev() {
				if dealNums > 2 {
					return
				}
			}
			s.YieldChan <- userMoney
		}

		if len(list) >= limit {
			lastId = list[limit-1].Id
		} else {
			return
		}
	}
}

func workFunc(s *scheduler.Scheduler) {
	for {
		select {
		case userMoney, ok := <-s.YieldChan:
			if !ok {
				//已关闭
				dlog.Info("worker Name", s.WorkerName)
				return
			}
			//查询record
			switch record := userMoney.(type) {
			case *table.UserMoney:
				clear, err := saveClearMoney(record)
				dlog.Info("worker Name", s.WorkerName, "unionId", record.QscUnionid, "pre_clear_money", clear, "err", err)
			}
		}
	}
}

```
