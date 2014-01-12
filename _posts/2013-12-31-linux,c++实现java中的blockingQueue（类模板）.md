---
layout: post
tag: linux C
category: linux-c
---
####Blocking Queue
<hr/>

	#include <pthread.h>
	#include <stdio.h>
	#include <iostream>
	#include <stdlib.h>
	#define DEFAULTSIZE 10
	using namespace std;
	template <class T>
	class blockingQueue
	{
			private:
					int size;
					int currentSize;
					bool empty;
					int readpos;
					int writepos;
					T *queue;
					pthread_mutex_t mutex;
					pthread_cond_t notempty;
					pthread_cond_t notfull;
			public:
					blockingQueue();
					blockingQueue(int s);
					void put(T &data);
					T get();
	};
	template<class T>
	blockingQueue<T>::blockingQueue()
	{
			blockingQueue(DEFAULTSIZE);
	}

	template<class T>
	blockingQueue<T>::blockingQueue(int s):size(s),currentSize(0),readpos(0),writepos(0)
	{
			queue=new T[s];
			pthread_mutex_init(&mutex,NULL);
			pthread_cond_init(&notempty,NULL);
			pthread_cond_init(&notfull,NULL);

	}

	template<class T>
	void blockingQueue<T>::put(T &data)
	{
			int ref=pthread_mutex_lock(&mutex);
			if(ref!=0)
			{
					perror("pthread_mutex_lock:");
					exit(-1);
			}
			if(currentSize==size)
			{
					cout<<"Queue is full"<<endl;
					pthread_cond_wait(&notfull,&mutex);
			}
			queue[writepos]=data;
			currentSize++;
			writepos++;
			if(writepos>=size)
					writepos=0;
			pthread_mutex_unlock(&this->mutex);
			pthread_cond_signal(&this->notempty);
	}

	template<class T>
	T blockingQueue<T>::get()
	{
			int ref=pthread_mutex_lock(&mutex);
			if(ref!=0)
			{
					perror("pthread_mutex_lock:");
					exit(-1);
			}
			T result;
			if(currentSize==0)
			{
					cout<<"Queue is empty"<<endl;
					pthread_cond_wait(&notempty,&mutex);
			}
			result=queue[readpos];
			readpos++;
			currentSize--;
			if(readpos>=size)
					readpos=0;
			pthread_mutex_unlock(&this->mutex);
			pthread_cond_signal(&this->notfull);
			return result;
	}
