## Fragment 懒加载简单封装 


```
package com.xiaoxian.tjsy.base;

import android.app.Dialog;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.LinearLayout;
import android.widget.RelativeLayout;

import com.xiaoxian.tjsy.R;
import com.xiaoxian.tjsy.utils.MyToast;

import butterknife.ButterKnife;

import static com.xiaoxian.tjsy.http.Constans.DIALOG_SHOW;

/**
 * Created by ziyang on 16/12/28.
 * Version 1.0
 * 封装Fragment懒加载
 * <p>
 * onCreateView() xml
 * onViewCreated() butterKnife initView()
 */

public abstract class BaseLazyFragment extends Fragment {

    private boolean isPrepared;
    private boolean isFirstVisible = true;
    private boolean isFirstInvisible = true;

    private Dialog mPro;
    protected MyToast myToast;
    protected Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            if (msg.what == DIALOG_SHOW) {
                mPro.show();
            } else {
                mPro.dismiss();
            }

        }
    };

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        int layoutId = getContentViewLayoutID();
        if (layoutId != 0) {
            return inflater.inflate(layoutId, container, false);
        } else {
            return super.onCreateView(inflater, container, savedInstanceState);
        }
    }

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        initPrepare();

        mPro = createLoadingDialog1(getActivity());
        myToast = new MyToast(getActivity());
    }

    private synchronized void initPrepare() {
        if (isPrepared) {
            onFirstUserVisible();
        } else {
            isPrepared = true;
        }
    }


    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        ButterKnife.bind(this, view);
        initViewsAndEvents(view);
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (isVisibleToUser) {
            if (isFirstVisible) {
                isFirstVisible = false;
                initPrepare();
            } else {
                onUserVisible();
            }
        } else {
            if (isFirstInvisible) {
                isFirstInvisible = false;
                onFirstUserInvisible();
            } else {
                onUserInvisible();
            }
        }
    }

    @Override
    public void onDestroy() {
        super.onDestroy();

        destroyViewAndThing();
    }

    protected abstract void onFirstUserVisible();//加载数据，开启动画/广播..

    protected abstract void onUserVisible();///开启动画/广播..

    private void onFirstUserInvisible() {
    }

    protected abstract void onUserInvisible();//暂停动画，暂停广播

    protected abstract int getContentViewLayoutID();

    protected abstract void initViewsAndEvents(View view);

    protected abstract void destroyViewAndThing();//销毁动作

    public Dialog createLoadingDialog1(Context context) {
        LayoutInflater inflater = LayoutInflater.from(context);
        View v = inflater.inflate(R.layout.loading_dialog, null);        // 得到加载view
        RelativeLayout layout = (RelativeLayout) v.findViewById(R.id.rl_probar);  // 加载布局
        Dialog loadingDialog = new Dialog(context, R.style.loading_dialog); // 创建自定义样式dialog
        layout.setVisibility(View.VISIBLE);
        loadingDialog.setCancelable(true);// 不可以用"返回键"取消
        loadingDialog.setCanceledOnTouchOutside(true);
        loadingDialog.setContentView(layout, new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT));
        return loadingDialog;
    }

    /**
     * 封装跳转
     */
    protected void readyGo(Class<?> clazz) {
        Intent intent = new Intent(getActivity(), clazz);
        startActivity(intent);
    }

    protected void readyGo(Class<?> clazz, Bundle bundle) {
        Intent intent = new Intent(getActivity(), clazz);
        if (null != bundle) {
            intent.putExtras(bundle);
        }
        startActivity(intent);
    }

    protected void readyGoForResult(Class<?> clazz, int requestCode) {
        Intent intent = new Intent(getActivity(), clazz);
        startActivityForResult(intent, requestCode);
    }

    protected void readyGoForResult(Class<?> clazz, int requestCode, Bundle bundle) {
        Intent intent = new Intent(getActivity(), clazz);
        if (null != bundle) {
            intent.putExtras(bundle);
        }
        startActivityForResult(intent, requestCode);
    }

}

```
